Robust af\_lib\_set\_attribute\*() Calls
========================================

All forms of `af_lib_set_attribute()` provide a return value to indicate whether the set request has been successfully enqueued. Checking this return value and re-trying can make your use of `af_lib_set_attribute*()` more robust. Here’s a code snippet from the afBlink example sketch that demonstrates some variations of this:

```
...SNIP...
bool rebootPending = false;     // True if reboot needed, e.g. if we received an OTA firmware
...SNIP...

// Callback executed any time ASR has information for the MCU
void attrEventCallback(const af\_lib\_event\_type\_t eventType,
            const af\_lib\_error\_t error,
            const uint16\_t attributeId,
            const uint16\_t valueLen,
            const uint8\_t\* value) {

  switch (eventType) {
    case AF\_LIB\_EVENT\_UNKNOWN:
      break;

...SNIP...

    case AF\_LIB\_EVENT\_ASR\_NOTIFICATION:
      // Unsolicited notification of non-MCU attribute change
      switch (attributeId) {

// EXAMPLE 1:
        case AF\_MODULO\_BUTTON:
          // curButtonValue is checked in loop(). If changed, will toggle blinking state.
          curButtonValue = \*(uint16\_t\*) value;
          break;

        case AF\_SYSTEM\_ASR\_STATE:
          switch (value\[0\]) {
            case AF\_MODULE\_STATE\_REBOOTED:
              break;

            case AF\_MODULE\_STATE\_LINKED:
              break;

            case AF\_MODULE\_STATE\_UPDATING:
              break;

EXAMPLE 2:
            case AF\_MODULE\_STATE\_UPDATE\_READY:
              // rebootPending is checked in loop(). If true, will send reboot command.
              rebootPending = true;
              break;

            case AF\_MODULE\_STATE\_INITIALIZED:
              break;

            default:
              break;
          }
          break;

        default:
          break;
      }
      break;

...SNIP...
  }
}

...SNIP...

void setModuloLED(bool on) {
  if (moduloLEDIsOn != on) {
    int16\_t attrVal = on ? LED\_ON : LED\_OFF; // Modulo LED is active low

// EXAMPLE 3:
    int timeout = 0;
    while (af\_lib\_set\_attribute\_16(af\_lib, AF\_MODULO\_LED, attrVal, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE) != AF\_SUCCESS) {
      delay(10);
      af\_lib\_loop(af\_lib);
      timeout++;
      if (timeout > 500) {
        // If we haven't been successful after 5 sec (500 tries, each after 10 msec delay) 
        // we assume we're in some desperate state, and reboot
        pinMode(RESET, OUTPUT);
        digitalWrite(RESET, 0);
        delay(250);
        digitalWrite(RESET, 1);
        return;
      }
    }

    moduloLEDIsOn = on;
  }
}

void loop() {
  // If we were asked to reboot (e.g. after an OTA firmware update), make the call here in loop().
  // In order to make this fault-tolerant, we'll continue to retry if the command fails.
  if (rebootPending) {
    int retVal = af\_lib\_set\_attribute\_32(af\_lib, AF\_SYSTEM\_COMMAND, AF\_MODULE\_COMMAND\_REBOOT, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE);
    rebootPending = (retVal != AF\_SUCCESS);
  }

  // Modulo button toggles 'blinking'
  if (prevButtonValue != curButtonValue) {
    if (af\_lib\_set\_attribute\_bool(af\_lib, AF\_BLINK, !blinking, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE) == AF\_SUCCESS) {
      blinking = !blinking;
      prevButtonValue = curButtonValue;
    }
  }

  // Give the afLib state machine some time.
  af\_lib\_loop(af\_lib);
}
```

**Notes:**

*   In all three examples, we check the return value from `af_lib_set_attribute()`, and retry if the call was not successful. Doing this is essential to robust afLib usage.
    
*   While retrying unsuccessful afLib calls, it is important to call `af_lib_loop()` to ensure that afLib gets time to handle requests. If, for example, we called `set_attribute()` and the call failed due to a full request queue, retrying the call would be pointless unless afLib got time to service the queue.
    
*   It’s important to avoid calling `af_lib_loop()` within `attrEventCallback()`; doing so can have unexpected results. But we know that to make afLib calls robust we should confirm-and-retry, and we know that we must call `af_lib_loop()` _while_ we retry. So if the callback tells us we need to call set\_attribute, how do we do that robustly?
    
    Examples #1 and #2 demonstrate a useful pattern: Code in the callback is restricted to setting a flag to indicate a `af_lib_set_attribute()` call is required. Then in the main `loop()`, the flag is checked, `set_attribute()` is called if indicated, and retried as needed until success. This pattern is robust, easy to read, and can reduce redundant code.
    
*   Example #3 shows a variation in which retrying is limited by a timeout: as above, the code retries `af_lib_set_attribute()`, waiting for AF\_SUCCESS. But here a timeout prevents an indefinite cycle of re-trying in the face of some serious condition that is blocking us. If the timeout is exceeded, we assume that communication with afLib is fatally obstructed, so we trigger a reboot by directly manipulating the reset pin.
    

Next: [Useful Debugging Methods](DebugMethods)

Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()
