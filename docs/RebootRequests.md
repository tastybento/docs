Handling Reboot Requests
========================

Your MCU code will use the `attrEventCallback()` callback to take action when attribute values change. Your primary concern will be changes to attributes you define, but your code must also watch for changes to the system attribute AF\_SYSTEM\_ASR\_STATE. This attribute can have one of several values, including:

*   Rebooted
*   Linked
*   Updating
*   Update Ready
*   Initialized
*   Relinked
*   Factory Reset

AF\_SYSTEM\_ASR\_STATE will change to “Updating” whenever ASR receives an Over-the-Air (OTA) firmware or profile update. When the OTA is complete, ASR will change AF\_SYSTEM\_ASR\_STATE to “Update Ready”, and your MCU code is responsible for recognizing this case and triggering a reboot.

Your code should initiate a controlled (“soft”) reboot by calling `af_lib_set_attribute()` for the attribute AF\_SYSTEM\_COMMAND, to the value AF\_MODULE\_COMMAND\_REBOOT. Here’s a snippet:

```
bool initializationPending = true;  // If true, we're waiting on AF\_MODULE\_STATE\_INITIALIZED
bool rebootPending = false;          // If true, a reboot is needed (e.g. if we received an OTA firmware update.)

// SNIP //

void attrEventCallback(const af\_lib\_event\_type\_t eventType,
                       const af\_lib\_error\_t error,
                       const uint16\_t attributeId,
                       const uint16\_t valueLen,
                       const uint8\_t\* value) {

  switch (attributeId) {
// SNIP //

    case AF\_SYSTEM\_ASR\_STATE:
      switch (value\[0\]) {

    case AF\_MODULE\_STATE\_REBOOTED:
      // ASR has been rebooted
      initializationPending = true;
      break;

    case AF\_MODULE\_STATE\_LINKED:
      break;

    case AF\_MODULE\_STATE\_UPDATING:
      break;

    case AF\_MODULE\_STATE\_UPDATE\_READY:
      // ASR signals that an update has been received and a reboot is required to use it
      rebootPending = true;
      break;

    case AF\_MODULE\_STATE\_INITIALIZED:
      // ASR signals that it's ready
      initializationPending = false;
      break;

    case AF\_MODULE\_STATE\_RELINKED:
      break;

    case AF\_MODULE\_STATE\_FACTORY\_RESET:
      // ASR signals that is has been factory reset; MCU should handle any local housekeeping and then reboot ASR
      rebootPending = true;
      break;

    default:
      break;
    }
    break;

  default:
    break;
  }
}

void loop() {
  af\_lib\_loop(af\_lib);    // Give the afLib state machine some time.

  if (initializationPending) {
    // If we're awaiting initialization, don't bother checking/setting attributes
  } else {
    
    if (rebootPending) {  // Someone wants us to reboot
      int retVal = af\_lib\_set\_attribute\_32(af\_lib, AF\_SYSTEM\_COMMAND, AF\_MODULE\_COMMAND\_REBOOT, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE);
      rebootPending = (retVal != AF\_SUCCESS);

      // Check for success; if not successful, we'll re-try next time around loop()
      if (!rebootPending) {
        // Reboot command sent; now awaiting AF\_MODULE\_STATE\_INITIALIZED
        initializationPending = true;
      }

    }

  // Your application code here //
  
  }
}
```
Our `attrEventCallback()` checks the supplied attribute ID, looking for AF\_SYSTEM\_ASR\_STATE. If we have received an update message for that attribute ID, we check the attribute value. If the value is 3 (AF\_MODULE\_STATE\_UPDATE\_READY), then we trigger a reboot by setting a global, `rebootPending`. In our main `loop()` function, any time `rebootPending` is true, we’ll call `af_lib_set_attribute_32()` for the AF\_SYSTEM\_COMMAND attribute, with value 1 (which is the value that signals a reboot). Our code resets `rebootPending` to false if it succeeds; if the `set_attribute()` call fails (for instance if the request queue is full), we’ll try again next time around the `loop()`.

Next: [Robust af\_lib\_set\_attribute\*() Calls](RobustafLibSet)

Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()
