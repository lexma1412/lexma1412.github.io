---
layout: default
title:  "UDS Service"
---

# UDS SERVICE
This post will introduce some UDS services that are used frequently in automotive.

## Read DTC Information - SID=0x19
Purpose of this service is to read DTC including snapshot and extended data<br />
This service has a sub-function byte with range from 0x00...0x7F. The definition of each value is defined in ISO14229-1 Table 269. I highlight some subfunction bytes that are important to me.

| Sub function byte | Description |
|:---|:---|
| 0x01 | **reportNumberOfDTCByStatusMask**<br />This parameter specifies that the server shall transmit to the client the number of DTCs matching a client-defined status mask. |
| 0x02 | **reportDTCByStatusMask**<br />This parameter specifies that the server shall transmit to the client a list of DTCs and corresponding statuses matching a client-defined status mask. |
| 0x04 | **reportDTCSnapshotRecordByDTCNumber**<br />This parameter specifies that the server shall transmit to the client the DTCSnapshot record(s) associated with a client-defined DTC number and DTCSnapshot record number (0xFF for all records). |
| 0x06 | **reportDTCExtDataRecordByDTCNumber**<br />This parameter specifies that the server shall transmit to the client the DTCExtendedData record(s) associated with a client-defined DTC number and DTCExtendedData record number (0xFF for all records, 0xFE for all OBD records). |

| Definition |
|:---|
| **DTCMaskRecord**: 3-byte value containing DTCHighByte, DTCMiddleByte and DTCLowByte, which together represent a unique identification number for a specific diagnostic trouble code (DTC) supported by a server. |
| **DTCSnapshotRecordNumber**: 1-byte value indicating the number of the specific DTCSnapshot data record requested for a client-defined DTCMaskRecord, specified by OEM. |
| **DTCExtDataRecordNumber**: 1-byte value indicating the number of the specific DTCExtendedData record requested for a client-defined DTCMaskRecord via the reportDTCExtDataRecordByDTCNumber and reportDTCExtDataRecordByRecordNumber sub-function. |
| **statusOfDTC**: The status of a particular DTC. Bits that are not supported by the server shall be reported as '0'. |

### SID=0x19, Subfunction=0x04
When Subfunction=0x04, SID 0x19 is used to read Snapshot.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x19 |
| 2 | SubfunctionByte | 0x04 |
| 3 | DTCHighByte | 0x00-0xFF |
| 4 | DTCMiddleByte | 0x00-0xFF |
| 5 | DTCLowByte | 0x00-0xFF |
| 6 | DTCSnapshotRecordNumber | 0x00-0xFF |

1. DTCMaskRecord: If request data is just snapshot data (means it does not relate to any DTC, it is just the data collected at one point in time, e.g. timestamp, temperature, ignition, odometer, etc.), these bytes can be ignored (set to any value).
2. DTCSnapshotRecordNumber: when DTCSnapshotRecordNumber=0xFF, the server reports all stored DTCSnapshot data records at once.

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x59 |
| 2 | SubfunctionByte | 0x04 |
| 3 | DTCHighByte | 0x00-0xFF |
| 4 | DTCMiddleByte | 0x00-0xFF |
| 5 | DTCLowByte | 0x00-0xFF |
| 6 | statusOfDTC | 0x00-0xFF |
| 7 | DTCSnapshotRecordNumber#1 | 0x00-0xFF |
| 8 | DTCSnapshotRecordNumberOfIdentifiers#1 | 0x00-0xFF |
| 9 | dataIdentifier#1 byte#1 (MSB) | 0x00-0xFF |
| 10 | dataIdentifier#1 byte#2 (LSB) | 0x00-0xFF |
| 11 | snapshotData#1 byte#1 | 0x00-0xFF |
| 11+(n-1) | snapshotData#1 byte#n | 0x00-0xFF |

* **Negative Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | Negative response | 0x7F |
| 2 | Request SID | 0x19 |
| 3 | Response code | 0x00-0xFF |

### SID=0x19, Subfunction=0x06
When Subfunction=0x06, SID 0x19 is used to read Extended data.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x19 |
| 2 | SubfunctionByte | 0x06 |
| 3 | DTCHighByte | 0x00-0xFF |
| 4 | DTCMiddleByte | 0x00-0xFF |
| 5 | DTCLowByte | 0x00-0xFF |
| 6 | DTCExtDataRecordNumber | 0x00-0xFF |

DTCExtDataRecordNumber: specified by OEM. When DTCExtDataRecordNumber=0xFF, the server reports all stored DTCExtendedData records in a single response message.<br>

In another post, I will discuss Extended data: Occurrence counter, Aging counter, Aged counter, Severity.

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x59 |
| 2 | SubfunctionByte | 0x06 |
| 3 | DTCHighByte | 0x00-0xFF |
| 4 | DTCMiddleByte | 0x00-0xFF |
| 5 | DTCLowByte | 0x00-0xFF |
| 6 | statusOfDTC | 0x00-0xFF |
| 7 | DTCExtDataRecordNumber#1 | 0x01 |
| 8 | Extended data record Byte#1 | 0x00-0xFF |
| 9 | DTCExtDataRecordNumber#n | 0x00-0xFF |
| 10 | Extended data record Byte#n | 0x00-0xFF |

* **Negative Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | Negative response | 0x7F |
| 2 | Request SID | 0x19 |
| 3 | Response code | 0x00-0xFF |

## Read Data by Identifier - SID=0x22
This service is used to read DID.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x22 |
| 2 | DataIdentifierByte#1 (MSB) | 0x00-0xFF |
| 3 | DataIdentifierByte#2 (LSB) | 0x00-0xFF |
| n-1 | DataIdentifierByte#n (MSB) | 0x00-0xFF |
| n | DataIdentifierByte#n (LSB) | 0x00-0xFF |

- In real projects, service 0x22 limits the number of DIDs that can be simultaneously requested, e.g. n=1 means only one DID can be requested at one time.

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x62 |
| 2 | DataIdentifierByte#1 (MSB) | 0x00-0xFF |
| 3 | DataIdentifierByte#2 (LSB) | 0x00-0xFF |
| 4~n | dataRecord | 0x00-0xFF |

## Diagnostic Session Control - SID=0x10
This service is used to request different diagnostic sessions. Basically, ISO14229-1 defines 2 basic sessions: default session and other sessions. The number of services that can be requested in each session (default or other) is defined in ISO14229.
1. Default session: server enters this session by default when powered up.
2. Other sessions: ProgrammingSession, extendedDiagnosticSession, and others defined by OEM.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x10 |
| 2 | diagnosticSessionType | 0x00-0xFF |

## Tester Present - SID=0x3E
This service is used to indicate to a server (or servers) that a client is still connected to the vehicle and that certain diagnostic services and/or communication that have been previously activated are to remain active.
This service is used to keep one or multiple servers in a diagnostic session other than the defaultSession. This can either be done by transmitting the TesterPresent request message periodically or in the absence of other diagnostic services to prevent the server(s) from automatically returning to the defaultSession.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x3E |
| 2 | zeroSubFunction | 0x00/0x80 |

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x3E |
| 2 | zeroSubFunction | 0x00 |

## Routine Control - SID=0x31
The RoutineControl service is used by the client to execute a defined sequence of steps and obtain any relevant results. E.g.: it may require the result of checking pre-conditions or request performing a self-test. In particular, in the first case, it might be necessary to switch the server into a specific diagnostic session using the DiagnosticSessionControl service or to unlock the server using the SecurityAccess service prior to using the StartRoutine service.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x31 |
| 2 | routineControlType | 0x01, 0x02, 0x03 |
| 3 | routineIdentifierByte#1 (MSB) | 0x00-0xFF |
| 4 | routineIdentifierByte#2 (LSB) | 0x00-0xFF |
| 5..n | routineControlOptionRecord | 0x00-0xFF |

routineIdentifier: ID of routine, specified by OEM<br />
routineControlType: 0x01, 0x02, 0x03 is start/stop/routine result accordingly.<br />
routineControlOptionRecord: this is optional, routine control may only run or stop when some precondition is satisfied; this information is requested via routineControlOptionRecord.

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x31 |
| 2 | routineControlType | 0x01, 0x02, 0x03 |
| 3 | routineIdentifierByte#1 (MSB) | 0x00-0xFF |
| 4 | routineIdentifierByte#2 (LSB) | 0x00-0xFF |
| 5 | routineInfo | 0x00-0xFF |

routineInfo: OEM specific and provides a mechanism for the vehicle manufacturer to support generic external test equipment handling of all implemented routines.

## Security Access - SID=0x27
This service provides a means to access data and/or diagnostic services, which have restricted access for security, emissions, or safety reasons. Diagnostic services for downloading/uploading routines or data into a server and reading specific memory locations from a server are situations where security access may be required.<br/>

Security Access service cannot be accessed in default session, therefore, Diagnostic Session Control - SID=0x10 shall be called before if needed.<br/>

The sequence of communication between client/server with Security Access service:<br/>

**Client**-----------------------------**Server**<br/>
Request securitySeed-------------><br/>
<----------------------------------securitySeed<br/>
securityKey------------------------><br/>

Server will verify securityKey. If valid, the service will be unlocked with the corresponding security level. If the client requests the same security level next time, the server will respond positively without securitySeed.

The popular algorithm used to authenticate and access security in UDS is Hash-based message authentication code (HMAC). (Hash algorithm can be HMAC_MD5, HMAC_SHA1, HMAC_SHA256, etc.) Refer [3] for details of the HMAC algorithm. Please note that securityKey here is not the key of the HMAC algorithm; we can understand that securitySeed is the message to be encrypted and securityKey is the HMAC output from the algorithm. Moreover, HMAC is compared by the sender message (Server), not the receiver (Client).

* **Request frame format**

**Request seed from client to server**:

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | Request SID | 0x27 |
| 2 | requestSeed | 0x01-0x41 |
| 3 | securityAccessDataRecordByte1 | 0x00-0xFF |
| n | securityAccessDataRecordByten | 0x00-0xFF |

requestSeed: is odd number, each value corresponds with different levels of security defined by OEM.

**Send key from client to server**:

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | Request SID | 0x27 |
| 2 | sendKey | 0x02-0x42 |
| 3 | securityKeyByte1 | 0x00-0xFF |
| n | securityKeyByten | 0x00-0xFF |

sendKey: is even number (= corresponding requestSeed + 1), each value corresponds with different levels of security defined by OEM.

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | Request SID | 0x67 |
| 2 | securityAccessType | 0x00-0x7F |
| 3 | securitySeedByte1 | 0x00-0xFF |
| n | securitySeedByten | 0x00-0xFF |

securitySeed: this parameter does not need to be in response if server is already unlocked.

## Request Download - SID=0x34
The requestDownload service is used by the client to initiate a data transfer from the client to the server.

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x34 |
| 2 | DataFormatIdentifier | 0x00-0xFF |
| 3 | AddressAndLengthFormatIdentifier | 0x00-0xFF |
| 4 | MemoryAddress Byte #1 | 0x00-0xFF |
| n | MemoryAddress Byte #n | 0x00-0xFF |
| n+1 | MemorySize Byte #1 | 0x00-0xFF |
| n+m | MemorySize Byte #m | 0x00-0xFF |

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x74 |
| 2 | LengthFormatIdentifier | 0x00-0xF0 |
| 3 | maxNumberOfBlockLength Byte #1 | 0x00-0xFF |
| n | maxNumberOfBlockLength Byte #n | 0x00-0xFF |

## Transfer Data - SID=0x36

* **Request frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x36 |
| 2 | blockSequenceCounter | 0x00-0xFF |
| 3 | transferRequestParameterRecord#1 | 0x00-0xFF |
| n | transferRequestParameterRecord#n | 0x00-0xFF |

* **Response frame format**

| No. of bytes | Description | Byte value |
|:---|:---|:---|
| 1 | SID | 0x76 |
| 2 | blockSequenceCounter | 0x00-0xFF |

## REFERENCE
[1] ISO14229-1<br/>
[2] https://piembsystech.com/read-dtc-information-service-0x19-uds-protocol<br/>
[3] https://www.baeldung.com/cs/hash-vs-mac#:~:text=MAC%2C%20in%20turn%2C%20is%20an,as%20part%20of%20its%20algorithm<br/>

