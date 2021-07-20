# GNetPlus Protocol

## Basic Package

The GNetPlus protocol uses the digital system as Big-Endian.

### Package Structure

| Offset | Bytes | Type | Name      | Description                                                  |
| :----: | :---: | :--: | --------- | ------------------------------------------------------------ |
|   0    |   1   |  u8  | Header Id | Start of Header <br />01h: SOH GNetPlus Start of Header      |
|   1    |   1   |  u8  | Address   | Command Device Address (For RS485)<br />00: Broadcast (Any Device) |
|   2    |   1   |  u8  | Code      | Request Code (Host to Device)<br />Others: Request code<br /><br />Response Code (Device to Host)<br />06h: ACK <br />15h: NAK<br />11h: Event |
|   3    |   1   |  u8  | Length    | Data Length (N Bytes)                                        |
|   4    |   N   |  u8  | Data      | Request parameters / Response Data                           |
|  4+N   |   2   | u16  | CRC16     | CRC16 |

### NAK Package Structure
| Offset | Bytes | Type | Name       | Description                                                  |
| :----: | :---: | :--: | ---------- | ------------------------------------------------------------ |
|   0    |   1   |  u8  | Header Id  | Start of Header <br />01h: SOH GNetPlus Start of Header      |
|   1    |   1   |  u8  | Address    | Command Device Address (For RS485)<br />00: Broadcast (Any Device) |
|   2    |   1   |  u8  | Code       | Response Code (Device to Host)<br />15h: NAK                 |
|   3    |   1   |  u8  | Length     | 01h: 1 Byte error code                                       |
|   4    |   N   |  u8  | Error Code | Response error code                                          |
|  4+N   |   2   | u16  | CRC16      | CRC16                                                        |


### CRC16 Calculation

#### C Header (GNetPlus.h)

``` C
#define CRC16_PRESET                            (0xFFFF)
#define CRC16_POLYNOM                           (0xA001)
unsigned short GNetPlus_CRC16_Byte(unsigned short uCRC16, unsigned char bData);
unsigned short GNetPlus_CRC16_Bytes(unsigned short uCRC16, const unsigned char* pBuffer, int iLength);
```

#### C Source (GNetPlus.c)

``` C
#include "GNetPlus.h"

unsigned short GNetPlus_CRC16_Byte(unsigned short uCRC16, unsigned char bData)
{
    int i;
    uCRC16^=bData;
    for(i=0; i<8; i++)
    {
        if(uCRC16 & 1)
        {
            uCRC16=(uCRC16>>1)^CRC16_POLYNOM;
        }
        else
        {
            uCRC16=(uCRC16>>1);
        }
    }
    return uCRC16;
}

unsigned short GNetPlus_CRC16_Bytes(unsigned short uCRC16, const unsigned char* pBuffer, int iLength)
{
    while(iLength--)
    {
        uCRC16=GNetPlus_CRC16_Byte(uCRC16, (*pBuffer++));
    }
    return uCRC16;
}
```

### GNetPlus Implement

#### C Header (GNetPlus.h)

``` C
#define GNETPLUS_ADDRESS_ANY_DEVICE             (0)     // Broadcast
#define GNETPLUS_CRC16_LENGTH                   (2)
#define GNETPLUS_PACKAGE_HEADER_ID_LENGTH       (1)
#define GNETPLUS_PACKAGE_HEADER_LENGTH          (4)
#define GNETPLUS_PACKAGE_MIN_LENGTH             (GNETPLUS_PACKAGE_HEADER_LENGTH+GNETPLUS_CRC16_LENGTH)
#define GNETPLUS_PACKAGE_DATA_MAX_LENGTH        (255)
#define GNETPLUS_BUFFER_MAX_LENGTH              (GNETPLUS_PACKAGE_HEADER_LENGTH+GNETPLUS_PACKAGE_DATA_MAX_LENGTH+GNETPLUS_CRC16_LENGTH)
typedef struct
{
    unsigned short uLength;
    union
    {
        struct
        {
            unsigned char bHeaderId;
            unsigned char bAddress;
            unsigned char bCode;
            unsigned char bLength;
            unsigned char bDatas[GNETPLUS_PACKAGE_DATA_MAX_LENGTH];
        };
        unsigned char bBuffer[GNETPLUS_BUFFER_MAX_LENGTH];
    };
} GNetPlus_Package_t;
typedef void (*PFSendByte)(unsigned char bData);
void GNetPlus_SendPackage(PFSendByte pfSendByte, unsigned char bAddress, unsigned char bCode, const unsigned char* pbDatas, unsigned char bLength);
void GNetPlus_MakePakge(GNetPlus_Package_t* ptPakcage, unsigned char bAddress, unsigned char bCode, const unsigned char* pbDatas, unsigned char bLength);
```

#### C Source (GNetPlus.c)

``` C
#include <string.h>
#include "GNetPlus.h"

#define ASCII_SOH                               (0x01)
void GNetPlus_SendPackage(PFSendByte pfSendByte, unsigned char bAddress, unsigned char bCode, const unsigned char* pbDatas, unsigned char bLength)
{
    unsigned short uCRC16;
    pfSendByte(ASCII_SOH);
    pfSendByte(bAddress);
    uCRC16=GNetPlus_CRC16_Byte(CRC16_PRESET, bAddress);
    pfSendByte(bCode);
    uCRC16=GNetPlus_CRC16_Byte(uCRC16, bCode);
    pfSendByte(bLength);
    uCRC16=GNetPlus_CRC16_Byte(uCRC16, bLength);
    if(0<bLength)
    {
        unsigned char bData;
        int i;
        for(i=0; i<bLength; i++)
        {
            bData=pbDatas[i];
            pfSendByte(bData);
            uCRC16=GNetPlus_CRC16_Byte(uCRC16, bData);
        }
    }
    pfSendByte((unsigned char)(uCRC16>>8));
    pfSendByte((unsigned char)uCRC16);
}

void GNetPlus_MakePakge(GNetPlus_Package_t* ptPakcage, unsigned char bAddress, unsigned char bCode, const unsigned char* pbDatas, unsigned char bLength)
{
    unsigned short uCRC16;
    memset(ptPakcage, 0, sizeof(GNetPlus_Package_t));
    ptPakcage->bHeaderId=ASCII_SOH;
    ptPakcage->bAddress=bAddress;
    ptPakcage->bCode=bCode;
    ptPakcage->bLength=bLength;
    if(0<bLength)
    {
        memcpy(ptPakcage->bDatas, pbDatas, bLength);
    }
    ptPakcage->uLength=(GNETPLUS_PACKAGE_MIN_LENGTH+bLength);
    uCRC16=GNetPlus_CRC16_Bytes(CRC16_PRESET,
                          &ptPakcage->bAddress,
                          (GNETPLUS_PACKAGE_HEADER_LENGTH-GNETPLUS_PACKAGE_HEADER_ID_LENGTH+bLength));
    ptPakcage->bDatas[bLength]=((unsigned char)(uCRC16>>8));
    ptPakcage->bDatas[bLength+1]=((unsigned char)uCRC16);
}
```

### Test Code

``` C
#include <stdio.h>
#include <string.h>
#include "GNetPlus.h"

#define UM800L_REQUEST_CODE_GET_VERSION         (0x10)
#define UM800L_VERSION_TYPE_FIRMWARE            (0)
#define UM800L_VERSION_TYPE_HARDWARE            (1)
void UM800L_Make_GetVersionPakcage(GNetPlus_Package_t* ptPakcage, unsigned char bVertionType)
{
    GNetPlus_MakePakge(ptPakcage, GNETPLUS_ADDRESS_ANY_DEVICE, UM800L_REQUEST_CODE_GET_VERSION, &bVertionType, 1);
}

void UM800L_Send_GetVersionPakcage(PFSendByte pfSendByte, unsigned char bVertionType)
{
    GNetPlus_SendPackage(pfSendByte, GNETPLUS_ADDRESS_ANY_DEVICE, UM800L_REQUEST_CODE_GET_VERSION, &bVertionType, 1);
}

void ShowPackage(const char* pszTitle, GNetPlus_Package_t* ptPackage)
{
    unsigned short i;
    printf("%s[%0u]: ", pszTitle, ptPackage->uLength);
    for(i=0; i<ptPackage->uLength; i++)
    {

        printf("%02X", ptPackage->bBuffer[i]);;
    }
    printf("\r\n");
}

GNetPlus_Package_t* m_ptPackage;

void SendByteToSaveToPackage(unsigned char bData)
{   // Call SendByte to output data to device e.g. UART.
    // The example is to save to the package
    if(m_ptPackage->uLength<GNETPLUS_BUFFER_MAX_LENGTH)
    {
        m_ptPackage->bBuffer[m_ptPackage->uLength++]=bData;
    }
}

int main()
{
    GNetPlus_Package_t tFWVersion1;
    GNetPlus_Package_t tFWVersion2;
    GNetPlus_Package_t tHWVersion1;
    GNetPlus_Package_t tHWVersion2;
    UM800L_Make_GetVersionPakcage(&tFWVersion1, UM800L_VERSION_TYPE_FIRMWARE);
    UM800L_Make_GetVersionPakcage(&tHWVersion1, UM800L_VERSION_TYPE_HARDWARE);
    m_ptPackage=&tFWVersion2;
    memset(&tFWVersion2, 0, sizeof(tFWVersion2));
    UM800L_Send_GetVersionPakcage(SendByteToSaveToPackage, UM800L_VERSION_TYPE_FIRMWARE);
    m_ptPackage=&tHWVersion2;
    memset(&tHWVersion2, 0, sizeof(tHWVersion2));
    UM800L_Send_GetVersionPakcage(SendByteToSaveToPackage, UM800L_VERSION_TYPE_HARDWARE);

    ShowPackage("Make Get F/W Package", &tFWVersion1);
    ShowPackage("Make Get H/W Package", &tHWVersion1);
    ShowPackage("Send Get F/W Package", &tFWVersion2);
    ShowPackage("Send Get H/W Package", &tHWVersion2);
    return 0;
}
```

### Test Code Output

``` Text
Make Get F/W Package[7]: 01001001007100
Make Get H/W Package[7]: 0100100101B1C1
Send Get F/W Package[7]: 01001001007100
Send Get H/W Package[7]: 0100100101B1C1
```

