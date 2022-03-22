Useful Debugging Methods
========================

This section includes a couple useful methods that you can copy/paste into your code. You can see these methods used in the afBlink example.

!!! note Due to limitations in available memory, if you’re working with an Arduino Uno, it’s not likely you’ll be able to use these methods fully. If it’s possible for you to use a board with more memory, such as a Teensy, you’ll have a great deal more freedom, including the use of these debug methods. If you are restricted to an Uno or system with similar available memory, you may be able to include a subset of these calls, and get more reliable performance by reducing the ATTR\_PRINT\_MAX\_VALUE\_LEN.

  
```
#define ATTR\_PRINT\_HEADER\_LEN     60
#define ATTR\_PRINT\_MAX\_VALUE\_LEN  512   // Each byte is 2 ASCII characters in HEX.
#define ATTR\_PRINT\_BUFFER\_LEN     (ATTR\_PRINT\_HEADER\_LEN + ATTR\_PRINT\_MAX\_VALUE\_LEN)

char attr\_print\_buffer\[ATTR\_PRINT\_BUFFER\_LEN\];

void getPrintAttrHeader(const char \*sourceLabel, const char \*attrLabel, const uint16\_t attributeId, const uint16\_t valueLen) {
    memset(attr\_print\_buffer, 0, ATTR\_PRINT\_BUFFER\_LEN);
    snprintf(attr\_print\_buffer, ATTR\_PRINT\_BUFFER\_LEN, "%s id: %s len: %05d value: ", sourceLabel, attrLabel, valueLen);
}

void printAttrBool(const char \*sourceLabel, const char \*attrLabel, const uint16\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    getPrintAttrHeader(sourceLabel, attrLabel, attributeId, valueLen);
    if (valueLen > 0) {
        strcat(attr\_print\_buffer, \*value == 1 ? "true" : "false");
    }
    Serial.println(attr\_print\_buffer);
}

void printAttr8(const char \*sourceLabel, const char \*attrLabel, const uint8\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    getPrintAttrHeader(sourceLabel, attrLabel, attributeId, valueLen);
    if (valueLen > 0) {
        char intStr\[6\];
        strcat(attr\_print\_buffer, itoa(\*((int8\_t \*)value), intStr, 10));
    }
    Serial.println(attr\_print\_buffer);
}

void printAttr16(const char \*sourceLabel, const char \*attrLabel, const uint16\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    getPrintAttrHeader(sourceLabel, attrLabel, attributeId, valueLen);
    if (valueLen > 0) {
        char intStr\[6\];
        strcat(attr\_print\_buffer, itoa(\*((int16\_t \*)value), intStr, 10));
    }
    Serial.println(attr\_print\_buffer);
}

void printAttr32(const char \*sourceLabel, const char \*attrLabel, const uint16\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    getPrintAttrHeader(sourceLabel, attrLabel, attributeId, valueLen);
    if (valueLen > 0) {
        char intStr\[11\];
        strcat(attr\_print\_buffer, itoa(\*((int32\_t \*)value), intStr, 10));
    }
    Serial.println(attr\_print\_buffer);
}

void printAttrHex(const char \*sourceLabel, const char \*attrLabel, const uint16\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    getPrintAttrHeader(sourceLabel, attrLabel, attributeId, valueLen);
    for (int i = 0; i < valueLen; i++) {
        strcat(attr\_print\_buffer, String(value\[i\], HEX).c\_str());
    }
    Serial.println(attr\_print\_buffer);
}

void printAttrStr(const char \*sourceLabel, const char \*attrLabel, const uint16\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    getPrintAttrHeader(sourceLabel, attrLabel, attributeId, valueLen);
    int len = strlen(attr\_print\_buffer);
    for (int i = 0; i < valueLen; i++) {
        attr\_print\_buffer\[len + i\] = (char)value\[i\];
    }
    Serial.println(attr\_print\_buffer);
}


void printAttribute(const char \*label, const uint16\_t attributeId, const uint16\_t valueLen, const uint8\_t \*value) {
    switch (attributeId) {
        case AF\_BLINK:
            printAttrBool(label, "AF\_BLINK", attributeId, valueLen, value);
            break;

        case AF\_MODULO\_LED:
            printAttr16(label, "AF\_MODULO\_LED", attributeId, valueLen, value);
            break;

        case AF\_GPIO\_0\_CONFIGURATION:
            printAttrHex(label, "AF\_GPIO\_0\_CONFIGURATION", attributeId, valueLen, value);
            break;

        case AF\_MODULO\_BUTTON:
            printAttr16(label, "AF\_MODULO\_BUTTON", attributeId, valueLen, value);
            break;

        case AF\_GPIO\_3\_CONFIGURATION:
            printAttrHex(label, "AF\_GPIO\_3\_CONFIGURATION", attributeId, valueLen, value);
            break;

        case AF\_BOOTLOADER\_VERSION:
            printAttrHex(label, "AF\_BOOTLOADER\_VERSION", attributeId, valueLen, value);
            break;

        case AF\_SOFTDEVICE\_VERSION:
            printAttrHex(label, "AF\_SOFTDEVICE\_VERSION", attributeId, valueLen, value);
            break;

        case AF\_APPLICATION\_VERSION:
            printAttrHex(label, "AF\_APPLICATION\_VERSION", attributeId, valueLen, value);
            break;

        case AF\_PROFILE\_VERSION:
            printAttrHex(label, "AF\_PROFILE\_VERSION", attributeId, valueLen, value);
            break;

        case AF\_SYSTEM\_ASR\_STATE:
            printAttr8(label, "AF\_SYSTEM\_ASR\_STATE", attributeId, valueLen, value);
            break;

        case AF\_SYSTEM\_LOW\_POWER\_WARN:
            printAttr8(label, "AF\_ATTRIBUTE\_LOW\_POWER\_WARN", attributeId, valueLen, value);
            break;

        case AF\_SYSTEM\_REBOOT\_REASON:
            printAttrStr(label, "AF\_REBOOT\_REASON", attributeId, valueLen, value);
            break;

        case AF\_SYSTEM\_LINKED\_TIMESTAMP:
            printAttr32(label, "AF\_SYSTEM\_LINKED\_TIMESTAMP", attributeId, valueLen, value);
            break;
    }
}
```

Here’s a brief example usage snippet:

```
void attrEventCallback(const af\_lib\_event\_type\_t eventType,
                       const af\_lib\_error\_t error,
                       const uint16\_t attributeId,
                       const uint16\_t valueLen,
                       const uint8\_t\* value) {
    printAttribute("attrEventCallback", attributeId, valueLen, value);
    // snip //
}
```
Next: [Handling MCU OTA Updates](MCU_OTA)

Updated July 30, 2021

  

© 2015-2021 Afero | [Legal](https://www.afero.io/html/home/privacy.html) | [Privacy](https://www.afero.io/html/home/privacy.html#privacy) | [Afero Home](https://www.afero.io)

[![Afero, Inc.](static/aflib/images/afero-logo.svg)]()
