---
layout: default
title:  "UDS Overview"
---

# UNIFIED DIAGNOSTIC SERVICE (UDS)
- In automotive industry, to analyze the vehicle when it on operation or when your car has problem and you get it to garage, we need to read diagnosis data retrieved inside ECU.
Therefore, the standardized protocol is created as UDS and defined in ISO 14229-1 to communicate with ECU.

## UDS with OSI layer
- According to [1], UDS is built base on 7 OSI layer, there are many related standards with low layer (transport, network, datalink, physical) that map with multi communication protocols (e.g: CAN, Flexray, LIN,..).
![UDS with OSL layer](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/OSI_Layer.png?raw=true)
- In this post, we focus on UDS using with CAN protocol only due to it is widely used in automotive world.

## UDS Request Frame format
UDS Request Frame is CAN frame, because we just focus on application layer, therefore we assume standard CAN 11 bit identifier is using in this post. For CAN protocol, you can refer to other post.
 ![UDS Request Frame](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_Request_Frame.png?raw=true)


### Service ID (SID)
ID of service, range from 0x00..0x3E for UDS ISO14229-1. The image below summarizes in detail list of SID defined in ISO14229-1.
![UDS Summary](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/SID_Summary.png?raw=true)

### Sub-function byte:
Some SIDs support sub-function that is a specific request action. 1st bit of Sub function byte has special meaning;
* 0‘ = FALSE: the ECU shall send a response, that is, no suppression of a positive response shall be done
* 1‘ = TRUE: Suppression of the positive response, that is, the ECU must not send a positive response
Negative responses shall be send by the ECU independent of this bit. 

### Request Data Parameters:
Request Data Parameters are including DID, DTC, Snapshot data, Extend data. 
*   DID is stand for **Data Identifier**, it usually is static data store in advance in ECU memory, it is read by SID=0x22(readDataByIdentifier) e.g: ECU Purchase Part Number. The definition list of DID can be found in Annex C-ISO14229-1. OEM can define their own DID support list base on standard.
*   DTC is stand for **Diagnostic Trouble Code**, DTC stores  event that satisfy error condition happening in ECU. DTC definition is not standardized, so it is defined specifically by OEM.
*   Snapshot data: it is a special DID collected at a specific point of time. Snapshot data is useful for diagnosing fault of vehicle since it can provide condition at the time fault occur. E.g: TimeStamp, Odometer, cabin temperature, Ignition status (CL15), Voltage Supply, .... In Autosar, it also called  **freeze frames**. To read snapshot data, using SID = 0x19 and subfunctionbyte = 4
*   Extend data: is a set of data which provides extended status information associated with a DTC. To read snapshot data, using SID = 0x19 and subfunctionbyte = 6. We will continue discuss the extend data in next post.

## UDS Response Frame format

![UDS Response Frame](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_Response_Frame.png?raw=true)

### Negative/Positive Response Service ID
*   In case sever/ECU is response normally, **Response SID = (Request SID + 0x40)**
*   In case error happen, e.g SID is not support, **Response SID = 0x7F**

### Response code
*   The response code only need in case negative response, the list of response code is defined in ISO14229-1. E.g: Responsecode = 0x12 mean subFunction Not Supported. Below image summarizes Common Response Code
![UDS Response Code](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/Response_Code.png?raw=true)

## UDS Simple Sequence
Below image demonstrates simple sequence for UDS
![UDS Simple Sequence](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_Sequence.png?raw=true)


### REFERENCE
[1] https://www.csselectronics.com/pages/uds-protocol-tutorial-unified-diagnostic-services#diagnostic-protocols
[2] ISO-14229-1