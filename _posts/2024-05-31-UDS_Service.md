---
layout: default
title:  "UDS Service"
---

# UDS SERVICE
This post will introduce some UDS services that used frequently in automotive.

## Read DTC Information-SID=0x19
Purpose of this service is to read DTC including snapshot and extend data<br />
This service has sub-function byte with range from 0x00... 0x7F, the definition of each value is defined in ISO14229-1 Table 269, I highlight some subfunction byte that important to me.

| Sub function byte | Description        |
|:----------------- |:------------------|
| 0x01              | **reportNumberOfDTCByStatusMask**<br />TThis parameter specifies that the server shall transmit to the client the number of DTCs matching a client defined status mask.|
| 0x02              | **reportDTCByStatusMask**<br />This parameter specifies that the server shall transmit to the client a list of  DTCs and corresponding statuses matching a client defined status mask.    |
| 0x04              | **reportDTCSnapshotRecordByDTCNumber**<br />This parameter specifies that the server shall transmit to the client the DTCSnapshot record(s) associated with a client defined DTC number and DTCSnapshot record number (0xFF for all records).     |
| 0x06              | **reportDTCExtDataRecordByDTCNumber**<br /> This parameter specifies that the server shall transmit to the client the DTCExtendedData record(s) associated with a client defined DTC number and DTCExtendedData record number (0xFF for all records, 0xFE for all OBD records).|

| Definition        |
|:----------------- |
|**DTCMaskRecord**:<br/> 3-byte value containing DTCHighByte, DTCMiddleByte and DTCLowByte, which together  represent a unique identification number for a specific diagnostic trouble code DTC supported by a server.|
|**DTCSnapshotRecordNumber**:<br/> 1-byte value indicating the number of the specific DTCSnapshot data record requested for a client defined DTCMaskRecord, therefore it is speicified by OEM|
|**DTCExtDataRecordNumber**:<br/> 1-byte value indicating the number of the specific DTCExtendedData record requested for a client defined DTCMaskRecord via the reportDTCExtDataRecordByDTCNumber and reportDTCExtDataRecordByRecordNumber sub-function|
|**statusOfDTC**:<br/> The status of a particular DTC, Bits that are not supported by the server shall be reported as '0'.|

### SID=0x19, Subfunction=0x04
* **Request frame format**

|No. of bytes	|Description	|Byte value|
|:--------------|:--------------|:---------|
|1	|SID	|0x19|
|2	|SubfunctionByte	|0x04|
|3	|DTCHighByte	|0x00–0xFF|
|4	|DTCMiddleByte	|0x00–0xFF|
|5	|DTCLowByte	|0x00–0xFF|
|6	|DTCSnapshotRecordNumber	|0x00–0xFF|

1. DTCMaskRecord: If request data is just snapshot data (mean does not related to any DTC, it just the data collected at one point of time, e.g. timestamp, temperature, ignition, odometer,..), these bytes can be ignored (set to any value).
1. DTCSnapshotRecordNumber: when DTCSnapshotRecordNumber=0xFF server to report all stored DTCSnapshot data records at once.


* **Respone frame format**

|No. of bytes	|Description	|Byte value|
|:--------------|:--------------|:---------|
|1	|SID	|0x59|
|2	|SubfunctionByte	|0x04|
|3	|DTCHighByte	|0x00–0xFF|
|4	|DTCMiddleByte	|0x00–0xFF|
|5	|DTCLowByte	|0x00–0xFF|
|6	|statusOfDTC	|0x00–0xFF|
|7	|DTCSnapshotRecordNumber#1|0x00–0xFF|
|8	|DTCSnapshotRecordNumberOfIdentifiers#1	|0x00~  0xFF|
|9	|dataIdentifier#1 byte#1(MSB)|0x00–0xFF|
|10	|dataIdentifier#1 byte#2 (LSB)|0x00–0xFF|
|11	|snapshotData#1 byte#1	
|11+(n-1)	|snapshotData#1 byte#n|0x00–0xFF|


* **Negative Respone frame format**

|No. of bytes   |Description    |Byte value|
|:--------------|:--------------|:---------|
|1	|negative response|	0x7F|
|2	|Request SID|	0x19|
|3	|Response code|0x00–0xFF|

### SID=0x19, Subfunction=0x06

* **Request frame format**

|No. of bytes	|Description	|Byte value|
|:--------------|:--------------|:---------|
|1	|SID	|0x19|
|2	|SubfunctionByte	|0x06|
|3	|DTCHighByte	|0x00–0xFF|
|4	|DTCMiddleByte	|0x00–0xFF|
|5	|DTCLowByte	|0x00–0xFF|
|6	|DTCExtDataRecordNumber	|0x00–0xFF|

DTCExtDataRecordNumber:  specified by OEM. When DTCExtDataRecordNumber=0xFF means the server to report all stored DTCExtendedData records in a single response message.<br>

In other post, I will discuss about Extended data: Occurence counter, Aging counter, Aged counter, Severity


* **Respone frame format**

|No. of bytes	|Description	|Byte value|
|:--------------|:--------------|:---------|
|1	|SID	|0x59|
|2	|SubfunctionByte	|0x06|
|3	|DTCHighByte	|0x00–0xFF|
|4	|DTCMiddleByte	|0x00–0xFF|
|5	|DTCLowByte	|0x00–0xFF|
|6	|statusOfDTC	|0x00-0xFF|
|7	|DTCExtDataRecordNumber#1	|0x01|
|8	|Extended data record Byte#1	|0x00-0xFF|
|9	|DTCExtDataRecordNumber#n	|0x00-0xFF|
|10	|Extended data record Byte#n	|0x00-0xFF|

* **Negative Respone frame format**

|No. of bytes   |Description    |Byte value|
|:--------------|:--------------|:---------|
|1	|Negative response|	0x7F|
|2	|Request SID|	0x19|
|3	|Response code|0x00–0xFF|


## Read Data by Identifier-SID=0x22

* **Request frame format**

|No. of bytes   |Description    |Byte value|
|:--------------|:--------------|:---------|
|1|SID	|0x22|
|2|DataIdentifierByte#1MSB	|0x00-0xFF|
|3|DataIdentifierByte#2LSB	|0x00-0xFF|
|n-1|DataIdentifierByte#nMSB	|0x00-0xFF|
|n|	DataIdentifierByte#nLSB	|0x00-0xFF|

- In real project, service 0x22 limit the number of  DID that can be simultaneously requested, e.g. n=1 means only one DID can be request at one time

* **Response frame format**

|No. of bytes   |Description    |Byte value|
|:--------------|:--------------|:---------|
|1	|SID|	0x62|
|2	|DataIdentifierByte#1MSB|	0x00-0xFF|
|3	|DataIdentifierByte#2LSB|	0x00-0xFF|
|4~n	|dataRecord|	0x00-0xFF|




## REFERENCE
https://piembsystech.com/read-dtc-information-service-0x19-uds-protocol/