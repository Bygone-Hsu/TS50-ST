# TS50-ST Command Set

## Table of Contents

1. History
2. [GNetPlus Protocol](GNetPlus%20Protocol.md)
    1. [Basic Package](GNetPlus%20Protocol.md#basic-package)
    2. [CRC16 Calculation](GNetPlus%20Protocol.md#crc16-calculation)
    3. [GNetPlus Implement](GNetPlus%20Protocol.md#gnetplus-implement)
3. [Commands](#3-commands)
    1. [Get Version](#3-1-get-version-command)
    2. [Device Control](#3-2-device-control-command)
4. [Error Code](#4-error-code)

## 3\. Commands

TS50-ST Commands is base on [GNetPlus Protocol](GNetPlus%20Protocol.md)

## *Request Code List*

| Name | Code | Meaning |
| :---: | :---: | ------- |
| [Get Version](#3-1-get-version-command) | `10h` | Get Firmware / Hardware version |
| [Device Control](#3-2-device-control-command) | `DCh` | Device I/O Control |

## 3-1\. Get Version Command

### Get Firmware/Hardware Version

* Parameter

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Index | Version Index<br>0: Firmware Version<br>1: Hardware Version |

* Response
    * NAK: Response an error code.
    * ACK: Success with response data
        * Response Data

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :--: | ------- | ---------------------------- |
| 0 | N | u8 | Version | Firmware or Hardware Version |

* Example
    * Get Firmware Version

<br />`[Send 7 Bytes] Get Version (10h Command)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | <div style='min-width:8em' align='center'>`ASCII`</div> |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `00` | `10` | `01` | `00` | `71` | `00` |  |  |  |  |  |  |  |  |  | `.....q.` |

| `Offset` | `Bytes` | <div style='min-width:12em' align='center'>`Data`</div> | <div style='min-width:32em' align='center'>`Description`</div> |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` 00` | `00h: Device Address: Broadcast (Any Device)` |
| `02` | `1` | ` 10` | `10h: Code: Get Version` |
| `03` | `1` | ` 01` | `Data Length` |
| `04` | `1` | ` 00` | `0: Version Index: Get Firmware Version` |
| `05` | `2` | ` 71 00` | `CRC16` |

<br />`[Receive 42 Bytes] Get Version Reply (ACK)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | <div style='min-width:8em' align='center'>`ASCII`</div> |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `FF` | `06` | `24` | `54` | `53` | `35` | `30` | `2D` | `53` | `54` | `20` | `52` | `4F` | `4D` | `2D` | `...$TS50-ST ROM-` |
| `10` | `54` | `32` | `30` | `32` | `34` | `20` | `56` | `31` | `2E` | `30` | `30` | `52` | `30` | `20` | `28` | `32` | `T2024 V1.00R0 (2` |
| `20` | `31` | `30` | `37` | `31` | `39` | `30` | `29` | `00` | `BD` | `5F` |  |  |  |  |  |  | `107190).._` |

| `Offset` | `Bytes` | <div style='min-width:12em' align='center'>`Data`</div> | <div style='min-width:32em' align='center'>`Description`</div> |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` FF` | `Device Address` |
| `02` | `1` | ` 06` | `06h: Code: ACK` |
| `03` | `1` | ` 24` | `Data Length` |
| `04` | `36` | ` 54 53 35 30 2D 53 54 20` <br /> ` 52 4F 4D 2D 54 32 30 32` <br /> ` 34 20 56 31 2E 30 30 52` <br /> ` 30 20 28 32 31 30 37 31` <br /> ` 39 30 29 00` | `Firmware Version:`<br />`TS50-ST ROM-T2024 V1.00R0 (2107190)` |
| `28` | `2` | ` BD 5F` | `CRC16` |

## 3-2\. Device Control Command

### Set Output with pattern

* Parameter

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Target Type | Control Target Type<br />0: GPIO |
| 1 | 1 | u8 | Control Function | Bit 0~1: Function<br />02h: Set<br /><br />Bit 2~7: Target<br />01h: Device, Special function GPIO, such as multi-color LED PIN or PWM, etc. |
| 2 | N | Structure | [Pattern] | [Output Pattern Structure](#output-pattern-structure) is repeatable |

* Response
    * NAK: Response an error code.
    * ACK: Success
* Example
    For the response of the example command, please refer to GNetPlus's example [Response an ACK package](GNetPlus%20Protocol.md#response-an-ack-package), [Response a NAK package](GNetPlus%20Protocol.md#response-a-nak-package)
    * GLED On + BLED Off (Infinite)
    <br />`[Send 24 Bytes] GLED On + BLED Off (Infinite) (DCh Command)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | <div style='min-width:8em' align='center'>`ASCII`</div> |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `00` | `DC` | `12` | `00` | `06` | `00` | `00` | `00` | `00` | `00` | `01` | `01` | `00` | `00` | `01` | `................` |
| `10` | `00` | `00` | `00` | `01` | `00` | `00` | `B7` | `92` |  |  |  |  |  |  |  |  | `........` |

| `Offset` | `Bytes` | <div style='min-width:12em' align='center'>`Data`</div> | <div style='min-width:32em' align='center'>`Description`</div> |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` 00` | `Device Address` |
| `02` | `1` | ` DC` | `DCh: Code: Device Control Command` |
| `03` | `1` | ` 12` | `Data Length` |
| `04` | `1` | ` 00` | `0: Control Target Type: GPIO` |
| `05` | `1` | ` 06` | `Bit 0~1: Function`<br />`02h: Set`<br /><br />`Bit 2~7: Target`<br />`01h: Device` |
| `06` | `1` | ` 00` | `00h: Repeat Option: By Count` |
| `07` | `1` | ` 00` | `00h: Device ID: LED0 (Red+Green LED)` |
| `08` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `0A` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `0B` | `1` | ` 01` | `1: Pattern Dot Count: 0` |
| `0C` | `1` | ` 01` | `1: LED0 Level: Green LED On` |
| `0D` | `1` | ` 00` | `0: Timeout: Infinite` |
| `0E` | `1` | ` 00` | `00h: Repeat Option: By Count` |
| `0F` | `1` | ` 01` | `01h: Device ID: LED1 (Blue LED)` |
| `10` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `12` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `13` | `1` | ` 01` | `1: Pattern Dot Count: 1` |
| `14` | `1` | ` 00` | `0: LED1 Level: Blue LED Off` |
| `15` | `1` | ` 00` | `0: Timeout: Infinite` |
| `16` | `2` | ` B7 92` | `CRC16` |

    * GLED+RLED Off (Infinite), BLED 200 ms Flash (Infinite)
<br />`[Send 26 Bytes] GLED+RLED Off (Infinite), BLED 200 ms Flash (Infinite) (DCh Command)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | <div style='min-width:8em' align='center'>`ASCII`</div> |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `00` | `DC` | `14` | `00` | `06` | `00` | `00` | `00` | `00` | `00` | `01` | `00` | `00` | `00` | `01` | `................` |
| `10` | `00` | `00` | `00` | `02` | `01` | `0A` | `00` | `0A` | `4A` | `53` |  |  |  |  |  |  | `........JS` |

| `Offset` | `Bytes` | <div style='min-width:12em' align='center'>`Data`</div> | <div style='min-width:32em' align='center'>`Description`</div> |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` 00` | `Device Address` |
| `02` | `1` | ` DC` | `DCh: Code: Device Control Command` |
| `03` | `1` | ` 14` | `Data Length` |
| `04` | `1` | ` 00` | `0: Control Target Type: GPIO` |
| `05` | `1` | ` 06` | `Bit 0~1: Function`<br />`02h: Set`<br /><br />`Bit 2~7: Target`<br />`01h: Device` |
| `06` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `07` | `1` | ` 00` | `00h: Device ID: LED0 (Red+Green LED)` |
| `08` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `0A` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `0B` | `1` | ` 01` | `1: Pattern Dot Count: 1` |
| `0C` | `1` | ` 00` | `0: LED0 Level: Green+Red LED Off` |
| `0D` | `1` | ` 00` | `0: Timeout: Infinite` |
| `0E` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `0F` | `1` | ` 01` | `01h: Device ID: LED1 (Blue LED)` |
| `10` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `12` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `13` | `1` | ` 02` | `2: Pattern Dot Count: 2` |
| `14` | `1` | ` 01` | `1: LED1 Level: Blue LED On` |
| `15` | `1` | ` 0A` | `0Ah: Timeout: 20 ms*10(0Ah)=200 ms` |
| `16` | `1` | ` 00` | `1: LED1 Level: Blue LED Off` |
| `17` | `1` | ` 0A` | `0Ah: Timeout: 20 ms*10(0Ah)=200 ms` |
| `18` | `2` | ` 4A 53` | `CRC16` |

    * GLED On + BLED Off (Infinite), GLED On 300 ms Off 160 ms once, Beep 300 ms once
<br />`[Send 44 Bytes] GLED On + BLED Off (Infinite), GLED On 300 ms Off 160 ms once, Beep 300 ms once (DCh Command)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | <div style='min-width:8em' align='center'>`ASCII`</div> |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `00` | `DC` | `26` | `00` | `06` | `00` | `00` | `00` | `00` | `00` | `01` | `01` | `00` | `00` | `01` | `...&............` |
| `10` | `00` | `00` | `00` | `01` | `00` | `00` | `00` | `00` | `00` | `01` | `00` | `02` | `01` | `0F` | `00` | `08` | `................` |
| `20` | `00` | `F0` | `00` | `01` | `00` | `02` | `01` | `0F` | `00` | `08` | `32` | `4A` |  |  |  |  | `..........2J` |

| `Offset` | `Bytes` | <div style='min-width:12em' align='center'>`Data`</div> | <div style='min-width:32em' align='center'>`Description`</div> |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` 00` | `Device Address` |
| `02` | `1` | ` DC` | `DCh: Code: Device Control Command` |
| `03` | `1` | ` 26` | `Data Length` |
| `04` | `1` | ` 00` | `0: Control Target Type: GPIO` |
| `05` | `1` | ` 06` | `Bit 0~1: Function`<br />`02h: Set`<br /><br />`Bit 2~7: Target`<br />`01h: Device` |
| `06` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `07` | `1` | ` 00` | `00h: Device ID: LED0 (Red+Green LED)` |
| `08` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `0A` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `0B` | `1` | ` 01` | `1: Pattern Dot Count: 1` |
| `0C` | `1` | ` 01` | `1: LED0 Level: Green On` |
| `0D` | `1` | ` 00` | `0: Timeout: Infinite` |
| `0E` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `0F` | `1` | ` 01` | `01h: Device ID: LED1 (Blue LED)` |
| `10` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `12` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `13` | `1` | ` 01` | `1: Pattern Dot Count: 1` |
| `14` | `1` | ` 00` | `1: LED1 Level: Blue LED Off` |
| `15` | `1` | ` 00` | `0: Timeout: Infinite` |
| `16` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `17` | `1` | ` 00` | `00h: Device ID: LED0 (Red+Green LED)` |
| `18` | `2` | ` 00 01` | `1: Repeat (By Count): 1` |
| `1A` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `1B` | `1` | ` 02` | `2: Pattern Dot Count: 2` |
| `1C` | `1` | ` 01` | `1: LED0 Level: Green On` |
| `1D` | `1` | ` 0F` | `0Fh: Timeout: 20ms*15(0Fh)=300 ms` |
| `1E` | `1` | ` 00` | `0: LED0 Level: Green+Red LED Off` |
| `1F` | `1` | ` 08` | `08h: Timeout: 20 ms*8(08h)=160 ms` |
| `20` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `21` | `1` | ` F0` | `F0h: Device ID: Buzzer` |
| `22` | `2` | ` 00 01` | `0: Repeat: Infinite` |
| `24` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `25` | `1` | ` 02` | `2: Pattern Dot Count: 2` |
| `26` | `1` | ` 01` | `1: Buzzer Level: On` |
| `27` | `1` | ` 0F` | `0Fh: Timeout: 20ms*15(0Fh)=300 ms` |
| `28` | `1` | ` 00` | `0: Buzzer Level: Off` |
| `29` | `1` | ` 08` | `08h: Timeout: 20 ms*8(08h)=160 ms` |
| `2A` | `2` | ` 32 4A` | `CRC16` |

    * GLED On + BLED Off (Infinite), RLED Flash 160 ms thrice, Beep On/Off 160 ms thrice
<br />`[Send 44 Bytes] GLED On + BLED Off (Infinite), RLED Flash 160 ms thrice, Beep On/Off 160 ms thrice (DCh Command)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | <div style='min-width:8em' align='center'>`ASCII`</div> |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `00` | `DC` | `26` | `00` | `06` | `00` | `00` | `00` | `00` | `00` | `01` | `01` | `00` | `00` | `01` | `...&............` |
| `10` | `00` | `00` | `00` | `01` | `00` | `00` | `00` | `00` | `00` | `03` | `00` | `02` | `02` | `08` | `00` | `08` | `................` |
| `20` | `00` | `F0` | `00` | `03` | `00` | `02` | `01` | `08` | `00` | `08` | `17` | `71` |  |  |  |  | `...........q` |

| `Offset` | `Bytes` | <div style='min-width:12em' align='center'>`Data`</div> | <div style='min-width:32em' align='center'>`Description`</div> |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` 00` | `Device Address` |
| `02` | `1` | ` DC` | `DCh: Code: Device Control Command` |
| `03` | `1` | ` 26` | `Data Length` |
| `04` | `1` | ` 00` | `0: Control Target Type: GPIO` |
| `05` | `1` | ` 06` | `Bit 0~1: Function`<br />`02h: Set`<br /><br />`Bit 2~7: Target`<br />`01h: Device` |
| `06` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `07` | `1` | ` 00` | `00h: Device ID: LED0 (Red+Green LED)` |
| `08` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `0A` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `0B` | `1` | ` 01` | `Pattern Dot Count` |
| `0C` | `1` | ` 01` | `1: LED0 Level: Green On` |
| `0D` | `1` | ` 00` | `0: Timeout: Infinite` |
| `0E` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `0F` | `1` | ` 01` | `01h: Device ID: LED1 (Blue LED)` |
| `10` | `2` | ` 00 00` | `0: Repeat: Infinite` |
| `12` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `13` | `1` | ` 01` | `1: Pattern Dot Count: 1` |
| `14` | `1` | ` 00` | `0: LED1 Level: Blue LED Off` |
| `15` | `1` | ` 00` | `0: Timeout: Infinite` |
| `16` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `17` | `1` | ` 00` | `00h: Device ID: LED0 (Red+Green LED)` |
| `18` | `2` | ` 00 03` | `0: Repeat: Infinite` |
| `1A` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `1B` | `1` | ` 02` | `2: Pattern Dot Count: 2` |
| `1C` | `1` | ` 02` | `2: LED0 Level: Red On` |
| `1D` | `1` | ` 08` | `08h: Timeout: 20 ms*8(08h)=160 ms` |
| `1E` | `1` | ` 00` | `0: LED0 Level: Green+Red LED Off` |
| `1F` | `1` | ` 08` | `08h: Timeout: 20 ms*8(08h)=160 ms` |
| `20` | `1` | ` 00` | `0: Repeat Option: By Count` |
| `21` | `1` | ` F0` | `F0h: Device ID: Buzzer` |
| `22` | `2` | ` 00 03` | `0: Repeat: Infinite` |
| `24` | `1` | ` 00` | `0: Dot Type: 1 Byte Pattern Dot` |
| `25` | `1` | ` 02` | `2: Pattern Dot Count: 2` |
| `26` | `1` | ` 01` | `1: Buzzer Level: On` |
| `27` | `1` | ` 08` | `08h: Timeout: 20 ms*8(08h)=160 ms` |
| `28` | `1` | ` 00` | `1: Buzzer Level: Off` |
| `29` | `1` | ` 08` | `08h: Timeout: 20 ms*8(08h)=160 ms` |
| `2A` | `2` | ` 17 71` | `CRC16` |

#### Repeat Option

Select the repetition method: By Count or By Timeout

| Value | Name       | Description                          |
| :---: | ---------- | ------------------------------------ |
|   0   | By Count   | Repeat the specified number of times |
|   1   | By Timeout | Repeat until timeout                 |

#### Output Pattern Structure
| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Option | Bit 0: [Repeat Option](#repeat-option)<br/>0: By Count<br/>1: By Timeout<br/><br/>Bit 1~7: RFU |
| 1 | 1 | u8 | Device ID | 00h: LED0 (Red+Green LED)<br />01h: LED1 (Blue LED)<br />F0h: Buzzer |
| 2 | 2 | u16 | Repeat | Repeat is the value of Big-Endian, [Repeat Option](#repeat-option) selects Repeat as Count or Timeout <br/>0: Infinite (Low Priority)<br />Note: Repeat not equal to 0 is high priority |
| 4 | 1 | u8 | Dot Option | Pattern Dot Option<br />Bit 0: Dot Type<br />0: [1 Byte Pattern Dot](#1-byte-pattern-dot-structure)<br />1: [2 Bytes Pattern Dot](#2-bytes-pattern-dot-structure) |
| 5 | 1 | u8 | Dot Count | Pattern Dot Count |
| 6 | N | Structure | [Pattern Dot] | Repeat [Pattern Dot Structure] for [Pattern Dot Count] times, and [Pattern Dot Option] selects Dot Structure as [1 Byte Pattern Dot Structure](#1-byte-pattern-dot-structure) or [2 Bytes Pattern Dot Structure](#2-bytes-pattern-dot-structure) |

#### 1 Byte Pattern Dot Structure

Structure can be repeated 12 times

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Level | On/Off Level |
| 1 | 1 | u8 | Timeout | Keep Level Timeout (Unit: 20ms) |

#### 2 Bytes Pattern Dot Structure

Structure can be repeated 6 times

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 2 | u16 | Level | On/Off Level (Big-Endian) |
| 2 | 2 | u16 | Timeout | Keep Level Timeout (Big-Endian, Unit: 1ms) |

## 4\. Error Code

| Value | Name | Description |
| :---: | ---- | ----------- |
| 00h | ERR NONE | No error occured |
| FFh | ERR UNKNOWN | Unknown error occured |
| FEh | ERR NOMEM | Not enough memory to perform the requested operation |
| FDh | ERR BUSY | Device or resource busy |
| FCh | ERR IO | Generic IO error |
| FBh | ERR TIMEOUT | Error due to timeout |
| FAh | ERR REQUEST | Invalid request or requested function can't be executed at the moment |
| F9h | ERR NOMSG | No message of desired type |
| F8h | ERR PARAM | Parameter error |
| F7h | ERR PROTO | Protocol error |
| F6h | ERR PROTO CRC | Protocol CRC16 check error |

