# TS50-ST Command Set

## Table of Contents

1. History
2. [GNetPlus Protocol](GNetPlus%20Protocol.md)
    1. [Basic Package](GNetPlus%20Protocol.md#basic-package)
    2. [CRC16 Calculation](GNetPlus%20Protocol.md#crc16-calculation)
    3. [GNetPlus Implement](GNetPlus%20Protocol.md#gntplus-implement)
3. [Commands](#3-commands)
    1. [Get Version](#3-1--get-version\(10h\)-command)
    2. [Device Control](#3-2--device-control\(dch\)-command)
4. [Error Code](#4-error-code)

## 3\. Commands

TS50-ST Commands is base on [GNetPlus Protocol](GNetPlus%20Protocol.md)

## *Request Code List*

| Name | Code | Meaning |
| :---: | :---: | ------- |
| [Get Version](#3-1-get-version\(10h\)-command) | `10h` | Get Firmware / Hardware version |
| [Device Control](#3-2-device-control\(dch\)-command) | `DCh` | Device I/O Control |

## 3\.1\. Get Version(10h) Command

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

`[Send 7 Bytes] Get Version (10h Command)`
| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | `ASCII` |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `00` | `10` | `01` | `00` | `71` | `00` |  |  |  |  |  |  |  |  |  | `.....q.` |

| `Offset` | `Bytes` | `Data` | `Description` |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` 00` | `Device Address` |
| `02` | `1` | ` 10` | `Function Code` |
| `03` | `1` | ` 01` | `Device Length` |
| `04` | `1` | ` 00` | `Version index`<br /> `0: Firmware`<br /> `1: Hardware` |
| `05` | `2` | ` 71 00` | `CRC16` |

`[Receive 42 Bytes] Get Version Reply (ACK)`

| `Offset` | `00` | `01` | `02` | `03` | `04` | `05` | `06` | `07` | `08` | `09` | `0A` | `0B` | `0C` | `0D` | `0E` | `0F` | `ASCII` |
| :------: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---- |
| `00` | `01` | `FF` | `06` | `24` | `54` | `53` | `35` | `30` | `2D` | `53` | `54` | `20` | `52` | `4F` | `4D` | `2D` | `...$TS50-ST ROM-` |
| `10` | `54` | `32` | `30` | `32` | `34` | `20` | `56` | `31` | `2E` | `30` | `30` | `52` | `30` | `20` | `28` | `32` | `T2024 V1.00R0 (2` |
| `20` | `31` | `30` | `37` | `31` | `39` | `30` | `29` | `00` | `BD` | `5F` |  |  |  |  |  |  | `107190).._` |

| `Offset` | `Bytes` | `Data` | `Description` |
| :------: | :---: | :--- | :---- |
| `00` | `1` | ` 01` | `SOH (Start of Heading)` |
| `01` | `1` | ` FF` | `Device Address` |
| `02` | `1` | ` 06` | `Reply Code` |
| `03` | `1` | ` 24` | `Device Length` |
| `04` | `37` | ` 54 53 35 30 2D 53 54 20` <br /> ` 52 4F 4D 2D 54 32 30 32` <br /> ` 34 20 56 31 2E 30 30 52` <br /> ` 30 20 28 32 31 30 37 31` <br /> ` 39 30 29 00 BD` | `Firmware Version:`<br /> `TS50-ST ROM-T2024 V1.00R0 (2107190)` |
| `29` | `2` | ` 5F 00` | `CRC16` |

## 3\.2\. Device Control(DCh) Command

### Set Output with pattern

* Parameter

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Target Type | Control Target Type<br />0: GPIO |
| 1 | 1 | u8 | Control Function | Bit 0~1: Function<br />02h: Set<br /><br />Bit 2~7: Target<br />01h: Device, 功能的GPIO, 複合多PIN或者PWM等特殊功能的GPIO |
| 2 | N | Structure | [Pattern] | [Output Pattern Structure](#output-pattern-structure)可以重覆 |

* Response
    * NAK: Response an error code.
    * ACK: Success

#### Repeat Option

選擇重覆的方式: 依次數或者依逾時

| Value | Name       | Description        |
| :---: | ---------- | ------------------ |
|   0   | By Count   | 重覆指定次數       |
|   1   | By Timeout | 重覆直到指定的逾時 |

#### Output Pattern Structure
| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Option | Bit 0: [Repeat Option](#repeat-option)<br/>0: By Count<br/>1: By Timeout<br/><br/>Bit 1~7: RFU |
| 1 | 1 | u8 | Device ID | 00h: LED0 (Red+Green LED)<br />01h: LED1 (Blue LED)<br />F0h: Buzzer |
| 2 | 2 | u16 | Repeat | Repeat為Big-Endian的數值, 依[Repeat Option](#repeat-option)決定Repeat為Count或者Timeout (單位: 1ms) |
| 4 | 1 | u8 | Dot Type | Bit 0: Pattern Dot Option<br />0: [1 Byte Pattern Dot](#1-byte-pattern-dot-structure)<br />1: [2 Bytes Pattern Dot](#2-bytes-pattern-dot-structure) |
| 5 | N | Structure | [Pattern Dot] | 依Pattern Dot Option決定Pattern Dot為[1 Byte Pattern Dot](#1-byte-pattern-dot-structure)或者[2 Bytes Pattern Dot](#2-bytes-pattern-dot-structure) |

#### 1 Byte Pattern Dot Structure

每一個Output Pattern可重覆12次的1 Byte Pattern Dot

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 1 | u8 | Level | On/Off Level |
| 1 | 1 | u8 | Timeout | Keep Level Timeout (單位: 20ms) |

#### 2 Bytes Pattern Dot Structure

每一個Output Pattern可重覆6次的2 Byte Pattern Dot

| Offset | Bytes | Type | Name | Description |
| :----: | :---: | :---: | ---- | ----------- |
| 0 | 2 | u16 | Level | On/Off Level (Big-Endian) |
| 2 | 2 | u16 | Timeout | Keep Level Timeout (Big-Endian, 單位: 1ms) |

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
| F6h | ERR PROTO CRC | Protocol error |
