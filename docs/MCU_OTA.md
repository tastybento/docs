Handling MCU OTA Updates
========================

In the [previous topic](RebootRequests), we looked at how ASR can receive an over-the-air (OTA) firmware update, and should be ready to handle the update by rebooting once the update is complete.

With Release 2.0 of the Afero firmware, we added a new feature: the ability to use the Afero OTA service to deliver not only updates of ASR firmware, but also updates to your MCU application code (or indeed any arbitrary file) running on developer devices. This page will describe what you’ll need to include in your application code to accept an MCU OTA update.

Overview of Handling an MCU OTA Update
--------------------------------------

At a high level, the steps involved in an MCU OTA update are as described below.

### Preparation

*   Use the Afero OTA Manager to [prepare a firmware update](OTAMgr) for your device. Note that the MCU OTA mechanism can be used to deliver any type of data to your device, not just MCU code.
    
*   Use the Afero Profile Editor to [define MCU OTA handling](AttrDef#MCUAttrs) for the device Profile involved.
    
    In order to support receipt of MCU OTA updates, your device Profile must include a few special attributes, including `AF_MCU_OTA_INFO` and `AF_MCU_OTA_TRANSFER`, which convey control information and the transferred data itself, respectively. All special attributes are added for you by the Afero Profile Editor when you select a particular firmware type to be received. Read more about this in the in the Profile Editor User Guide, [Configure the MCU](AttrDef#ConfigMCU) section.
    
*   You’ll include code in your MCU application to handle the messaging and data transfers involved in an MCU OTA update. The purpose of this page is to help you with that.

### OTA Delivery

*   You’ll use the Afero OTA Manager to [deploy a firmware image](OTAMgr#DeployFWImage) to your device.
*   The incoming update will be handled as described below. Your MCU code will:

*   Receive an AF\_LIB\_EVENT\_ASR\_NOTIFICATION event for attribute AF\_MCU\_OTA\_INFO.
*   Extract from that notification the incoming file type and amount of data to be received.
*   Respond that it is ready to receive the data.
*   Receive the data in the form of AF\_LIB\_EVENT\_ASR\_NOTIFICATION events for attribute AF\_MCU\_OTA\_TRANSFER.
*   For each data chunk received, respond with the amount of data received so far. When all expected data has been received, MCU will respond with AF\_MCU\_OTA\_STOP\_TRANSFER\_OFFSET.
*   Calculate a SHA-256 checksum of the data received, and send that checksum with the message AF\_OTA\_VERIFY\_SIGNATURE, which requests validation of the data.
*   Receive an AF\_LIB\_EVENT\_ASR\_NOTIFICATION event for attribute AF\_MCU\_OTA\_INFO, with either:
    
    *   AF\_OTA\_APPLY, indicating that the data transfer occurred correctly and the download can be used; or,
    *   AF\_OTA\_FAIL, indicating that the checksum did not match, and the downloaded data should be discarded.

### Example MCU OTA: Console Output

Below you’ll see some annotated console output showing an MCU OTA of an 8K test file. The output is from an instrumented form of the example code available at the bottom of this page; we suggest you look at this output and the code together.

```
[➀](#annote1)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrId AF\_MCU\_OTA\_INFO
calling handle\_ota\_info()

   handle\_ota\_info: ota\_info->state = AF\_OTA\_TRANSFER\_BEGIN
   handle\_ota\_info: about to get an MCU OTA for type 101 and size 8088
   handle\_ota\_info: would open file 'ota\_file\_type.101' to store MCU OTA
   handle\_ota\_info: Responding with an UPDATE to attribute id AF\_MCU\_OTA\_INFO, which tells ASR to start sending the OTA

[➁](#annote2)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrID AF\_MCU\_OTA\_TRANSFER
calling handle\_ota\_transfer()

   handle\_ota\_transfer: received this chunk: 249; total received so far: 249 of 8088 bytes, at 296.08 bytes/sec.
   handle\_ota\_transfer: Called setAttribute to report that we've received 249 bytes so far.

[➂](#annote3)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrId AF\_MCU\_OTA\_TRANSFER
calling handle\_ota\_transfer()

   handle\_ota\_transfer: received this chunk: 6; total received so far: 255 of 8088 bytes, at 21.82 bytes/sec.
   handle\_ota\_transfer: Called setAttribute to report that we've received 255 bytes so far.

[➃](#annote4)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrId AF\_MCU\_OTA\_TRANSFER
calling handle\_ota\_transfer()

   handle\_ota\_transfer: received this chunk: 249; total received so far: 504 of 8088 bytes, at 309.32 bytes/sec.
   handle\_ota\_transfer: Called setAttribute to report that we've received 504 bytes so far.

// SNIP - several chunk transfers omitted in the interest of brevity //

[➄](#annote5)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrId AF\_MCU\_OTA\_TRANSFER
calling handle\_ota\_transfer()

   handle\_ota\_transfer: received this chunk: 249; total received so far: 7921 of 8088 bytes, at 323.38 bytes/sec.
   handle\_ota\_transfer: Called setAttribute to report that we've received 7921 bytes so far.

[➅](#annote6)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrId AF\_MCU\_OTA\_TRANSFER
calling handle\_ota\_transfer()

   handle\_ota\_transfer: received this chunk: 167; total received so far: 8088 of 8088 bytes, at 276.49 bytes/sec.
   handle\_ota\_transfer: Looks like we're finished. Received: 8088 of 8088 bytes, at 287.10 bytes/sec.
   handle\_ota\_transfer: Called setAttribute with AF\_MCU\_OTA\_STOP\_TRANSFER\_OFFSET to say we're done.
   handle\_ota\_transfer: Finalize the SHA and package it for sending
   handle\_ota\_transfer: sending up SHA for file

[➆](#annote7)
attrEventCallback received AF\_LIB\_EVENT\_ASR\_NOTIFICATION with attrId AF\_MCU\_OTA\_INFO
calling handle\_ota\_info()

   handle\_ota\_info: ota\_info->state = AF\_OTA\_APPLY

➀
```
The console output begins when an MCU OTA update is triggered (externally) for this device; for example, by the Afero OTA Manager. At that point the MCU receives notification with the attribute ID = AF\_MCU\_OTA\_INFO, which is treated as a signal to call `handle_ota_info()`.

The attribute _value_ received by the callback is cast to a pointer to an `af_ota_info_t`, so we can access the fields by name. Within the `ota_info`, the `state` field is AF\_OTA\_TRANSFER\_BEGIN, which indicates that there’s an incoming OTA.

The `ota_info` also provides us with incoming data size, file name, and file type. In the example code, we track these details (and more) in the global `ota_state`, but that’s an implementation detail (and don’t confuse the global `ota_state` with the `ota_info->state`!). Track progress however you like; the only requirement is that you _must_ keep a running count of the bytes downloaded.

At this point, real-world code would probably open a file to save into; in the example we’ll just create a fake file pointer.

Once you’re ready to receive the data, you must call `af_lib_set_attribute_bytes()` for attribute AF\_MCU\_OTA\_INFO, including the same value and valueLen parameters you received in the first place. This signals ASR to start sending OTA update data.

If you do _not_ want the OTA to proceed after you’ve received the AF\_OTA\_TRANSFER\_BEGIN (e.g. if you attempted to open a file for the data, but that operation failed), then you should set the "state" field in the received "value" to AF\_OTA\_IDLE, and then call `af_lib_set_attribute_bytes()`.

  

➁

In response to our `set_attribute()` call signalling that the OTA should begin, we receive notification with the attribute ID = AF\_MCU\_OTA\_TRANSFER. This begins the phase during which the actual data transfer takes place; we handle it in `handle_ota_transfer()`.

The attribute value in AF\_MCU\_OTA\_TRANSFER notifications consists of the data being transferred (one variable-sized chunk at a time), prepended by 4 bytes indicating the offset into the full data payload represented by the current chunk.

After each chunk of data is received, the MCU must respond with how many bytes it has received. To do this, MCU calls `af_lib_set_attribute_bytes()` for attribute AF\_MCU\_OTA\_TRANSFER, with a value equal to the number of bytes of data received so far. When the number of bytes received equals the total bytes expected (i.e. when the transfer is complete), the `received_so_far` value should be set to AF\_MCU\_OTA\_STOP\_TRANSFER\_OFFSET, which signals that OTA update is complete.

As noted previously, the example code calculates the SHA-256 of the data as it is received. Calculation of the SHA is required, and can be done during the transfer, or once it is complete.

You will notice that the example code tracks the rate of data transfer–this is not required, but is included for demonstration purposes.

➂

Another chunk of data is received. Note that the amount of data per chunk can vary widely: in this case only 6 bytes came across. **Currently, the maximum chunk size is 253 bytes.**

➃

And another chunk. After this one, we snipped out several chunk-transfers, just for readability.

➄

Another chunk…we’re getting close to completion!

➅

When your code detects that all the expected data has been received, the MCU must request verification of the data it has received. To do this this, your code must include the SHA checksum and the state `AF_OTA_VERIFY_SIGNATURE` in an `af_ota_info_t` struct, and send that in a call to `af_lib_set_attribute_bytes()` for attribute `AF_MCU_OTA_INFO`.

➆

Finally: in response to the verification request, the MCU callback gets one more AF\_LIB\_EVENT\_ASR\_NOTIFICATION, in which the `ota_info->state` either confirms or denies the data was received correctly. If the state = AF\_OTA\_APPLY, the data SHA verified successfully. If the state is AF\_OTA\_FAIL, the SHA did not have the expected value, and the data should not be trusted, and your code will typically discard it.

At this point, the MCU OTA update is complete! Assuming the data was downloaded and verified, your MCU code should take whatever context-dependent steps are required to install and use it.

### Example Code to Handle an MCU OTA Update

The simplified sketch code below provides examples of handlers for AF\_MCU\_OTA\_INFO and AF\_MCU\_OTA\_TRANSFER events. This code was used to generate the console output shown on this page.

In order to accept and receive an MCU OTA update, your application code must:

*   Handle AF\_LIB\_EVENT\_ASR\_NOTIFICATION events for attribute AF\_MCU\_OTA\_INFO, with an attribute value that represents an `af_ota_info_t` struct (which is defined in `af_mcu_ota.h`). These AF\_MCU\_OTA\_INFO events inform your MCU code when an MCU OTA update is available, when it’s complete, and so on. The `af_ota_info_t` struct contains a lot of information, but one of the highest-level pieces is the `ota_state_t`. Your code should watch for, and react to the following OTA states:
    
    *   AF\_OTA\_IDLE
    *   AF\_OTA\_TRANSFER\_BEGIN
    *   AF\_OTA\_TRANSFER\_END
    *   AF\_OTA\_APPLY
    *   AF\_OTA\_FAIL
    
    In the example code below, this state-handling functionality is contained in the function `handle_ota_info()`.
    
*   Handle AF\_LIB\_EVENT\_ASR\_NOTIFICATION events for attribute AF\_MCU\_OTA\_TRANSFER.
    
    AF\_MCU\_OTA\_TRANSFER events represent the actual data transfer; the value in each of these messages is comprised of a chunk of data being sent, plus 4-byte prefix indicating the current offset into the total data payload.
    
    In the example code, we handle AF\_MCU\_OTA\_TRANSFER events in the function `handle_ota_transfer()`.
    
*   Calculate a SHA-256 checksum for the downloaded data. Once the download is complete this checksum must be sent to ASR, where it will be verified. Your code will receive confirmation/denial based upon this verification, and should not use the downloaded data unless the checksum is confirmed.
    
    In the example code, we calculate the SHA as data arrives because the example does not store the download; in real-world applications you are free to calculate the SHA as data arrives, or after the download is complete. The example code handles this task in `handle_ota_transfer()`.
    

The example code provides the skeleton for handling receipt of an incoming MCU OTA update, but does **not** include any procedures required to install, such as an OTA update once downloaded. Because such procedures will be platform-specific, writing that code is your responsibility.

  
```
typedef struct {
  af\_ota\_begin\_info\_t begin\_info;
  af\_ota\_apply\_info\_t apply\_info;
  char filename\[64\];
  int file;
  uint32\_t received\_so\_far;
  long transfer\_start\_time;
  long chunk\_start\_time;
} ota\_state\_t;

static ota\_state\_t ota\_state;
#define OTA\_FILE\_NAME\_PREFIX    "ota\_file\_type"

static isc\_sha256\_t sha256;                 // Used for our running SHA of incoming data
af\_ota\_info\_t sha\_ota\_info;                 // Context for reporting SHA

static void handle\_ota\_info(const uint16\_t valueLen, const uint8\_t\* value) {

  af\_ota\_info\_t\* ota\_info = (af\_ota\_info\_t\*) value;
  int res = AF\_SUCCESS;
  switch (ota\_info->state) {
    case AF\_OTA\_IDLE:
      break;

    case AF\_OTA\_TRANSFER\_BEGIN: {
      Serial.println("handle\_ota\_info: ota\_info->state = AF\_OTA\_TRANSFER\_BEGIN");
      memset(&ota\_state, 0, sizeof(ota\_state));
      ota\_state.begin\_info.ota\_type = af\_utils\_read\_little\_endian\_16((const uint8\_t\*) &ota\_info->info.begin\_info.ota\_type);
      ota\_state.begin\_info.size = af\_utils\_read\_little\_endian\_32((const uint8\_t\*) &ota\_info->info.begin\_info.size);

      Serial.print("handle\_ota\_info: about to get an MCU OTA for type ");
      Serial.print(ota\_state.begin\_info.ota\_type);
      Serial.print(" and size ");
      Serial.println(ota\_state.begin\_info.size);

      // Open a file to save the data we're about to get
      snprintf(ota\_state.filename, sizeof(ota\_state.filename), "%s.%d", OTA\_FILE\_NAME\_PREFIX, ota\_state.begin\_info.ota\_type);
      Serial.print("handle\_ota\_info: would open file '"); Serial.print(ota\_state.filename); Serial.println("' to store MCU OTA");

      // Example uses a fake file pointer--merely non-null value
      ota\_state.file = 1;
      isc\_sha256\_init(&sha256);

      // Responding with an UPDATE to attribute id AF\_MCU\_OTA\_INFO so ASR will start sending the OTA data
      res = af\_lib\_set\_attribute\_bytes(af\_lib, AF\_MCU\_OTA\_INFO, valueLen, value, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE);
    }
      break;

    case AF\_OTA\_TRANSFER\_END:
      Serial.println("handle\_ota\_info: ota\_info->state = AF\_OTA\_TRANSFER\_END");
      break;

    case AF\_OTA\_APPLY:
      Serial.println("handle\_ota\_info: ota\_info->state = AF\_OTA\_APPLY, which confirms SHA verified");
      // At this point app should take whatever action desired with downloaded data
      break;

    case AF\_OTA\_FAIL:
      Serial.println("handle\_ota\_info: ota\_info->state = AF\_OTA\_FAIL (SHA didn't verify)");
      break;

  }
}

static void handle\_ota\_transfer(const uint16\_t valueLen, const uint8\_t\* value) {

  // The format of this attribute is the first 4 bytes are the offset of the data and the rest are the actual data.
  // Updates to this attribute should just contain the next offset you want the data from (or AF\_MCU\_OTA\_STOP\_TRANSFER\_OFFSET to halt transfer)
  uint32\_t offset = af\_utils\_read\_little\_endian\_32(value);
  bool done = false;

  if (offset != ota\_state.received\_so\_far) {
    // Odd case: we're getting data for an offset that we don't expect. Respond with the correct value
    Serial.print("handle\_ota\_transfer: got offset ");
    Serial.print(offset);
    Serial.print(" but was expecting ");
    Serial.println(ota\_state.received\_so\_far);
  } else {
    uint32\_t amount\_to\_write = valueLen - sizeof(offset);

    ota\_state.received\_so\_far += amount\_to\_write;
    long now = af\_utils\_millis();
    float chunk\_speed = ((float) amount\_to\_write / (now - ota\_state.chunk\_start\_time)) \* 1000;

    // Update the SHA. In this example code, we don't save data, we just calculate SHA as data arrives.
    isc\_sha256\_update(&sha256, (uint8\_t\*) value + sizeof(offset)-1, amount\_to\_write);

    Serial.print("handle\_ota\_transfer: received this chunk: ");
    Serial.print(amount\_to\_write);
    Serial.print("; total received so far: ");
    Serial.print(ota\_state.received\_so\_far);
    Serial.print(" of ");
    Serial.print(ota\_state.begin\_info.size);
    Serial.print(" bytes, at ");
    Serial.print(chunk\_speed);
    Serial.println(" bytes/sec.");

    // Stop if we've received all data
    if (ota\_state.received\_so\_far >= ota\_state.begin\_info.size) {
      float total\_speed = ((float) ota\_state.begin\_info.size / (now - ota\_state.transfer\_start\_time)) \* 1000;
      Serial.print("handle\_ota\_transfer: Looks like we're finished. Received: ");
      Serial.print(ota\_state.received\_so\_far);
      Serial.print(" of ");
      Serial.print(ota\_state.begin\_info.size);
      Serial.print(" bytes, at ");
      Serial.print(total\_speed);
      Serial.println(" bytes/sec.");
      ota\_state.file = 0;
      ota\_state.received\_so\_far = AF\_MCU\_OTA\_STOP\_TRANSFER\_OFFSET;
      done = true;
    }
  }

  ota\_state.chunk\_start\_time = af\_utils\_millis();
  uint8\_t result\[sizeof(uint32\_t)\];
  af\_utils\_write\_little\_endian\_32(ota\_state.received\_so\_far, result);
  int res = af\_lib\_set\_attribute\_bytes(af\_lib, AF\_MCU\_OTA\_TRANSFER, sizeof(result), result, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE);
  if (res != AF\_SUCCESS) {
    Serial.print("handle\_ota\_transfer: error setting attribute ");
    Serial.print(AF\_MCU\_OTA\_TRANSFER);
    Serial.print("; got result ");
    Serial.println(res);
  } else {
    if (done) {
      Serial.print("handle\_ota\_transfer: Called setAttribute with AF\_MCU\_OTA\_STOP\_TRANSFER\_OFFSET to say we're done.");
    }
    else {
      Serial.print("handle\_ota\_transfer: Called setAttribute to report that we've received ");
      Serial.print(ota\_state.received\_so\_far);
      Serial.println(" bytes so far.");
    }
  }

// If we're done because we got all the data expected then send SHA to ASR to verify
  if (done) {
    Serial.println("Finalize the SHA and package it for sending");
    isc\_sha256\_final(sha\_ota\_info.info.verify\_info.sha, &sha256);

    Serial.println("handle\_ota\_transfer: sending up SHA for file");
    sha\_ota\_info.state = AF\_OTA\_VERIFY\_SIGNATURE;
    res = af\_lib\_set\_attribute\_bytes(af\_lib, AF\_MCU\_OTA\_INFO, sizeof(sha\_ota\_info), (uint8\_t \* ) & sha\_ota\_info, AF\_LIB\_SET\_REASON\_LOCAL\_CHANGE);
    if (res != AF\_SUCCESS) {
      Serial.print("handle\_ota\_transfer: error setting attribute: ");
      Serial.print(AF\_MCU\_OTA\_INFO);
      Serial.print("; err: ");
      Serial.println(res);
    }
  }
}

void attrEventCallback(const af\_lib\_event\_type\_t eventType,
                       const af\_lib\_error\_t error,
                       const uint16\_t attributeId,
                       const uint16\_t valueLen,
                       const uint8\_t\* value) {

  switch (eventType) {

    case AF\_LIB\_EVENT\_ASR\_NOTIFICATION:

      switch (attributeId) {
        case AF\_MCU\_OTA\_INFO:
          Serial.println("attrEventCallback received attribute AF\_MCU\_OTA\_INFO");
          handle\_ota\_info(valueLen, value);
          break;
        case AF\_MCU\_OTA\_TRANSFER:
          Serial.println("attrEventCallback received attribute AF\_MCU\_OTA\_TRANSFER");
          handle\_ota\_transfer(valueLen, value);
          break;
        case AF\_SYSTEM\_ASR\_STATE:
// SNIP //
          break;
      }
      break;
  }
}

void setup() {
// SNIP - the usual setup() //
}

void loop() {
//SNIP - the usual loop() //
}
```
Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()
