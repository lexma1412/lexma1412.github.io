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
    When Subfunction=0x04, SID 0x19 used to read Snapshot

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
    When Subfunction=0x06, SID 0x19 used to read Extend data

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
    This service used to read DID

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

    ## Diagnostic Session Control-SID=0x10
    This service used to request different diagnostic session. Basically, ISO14229-1 defines 2 basic sessions: default session and other session.The number of service can be request in each session (default or other) is defined in ISO14229.
    1. Default session: server default enter this session when power up
    1. Other session: ProgrammingSession, extendedDiagnosticSession and others defined by OEM

    * **Request frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID	|0x10|
    |2	|diagnosticSessionType	|0x00-0xFF|


    ## Tester Present-SID=0x3E
    This service is used to indicate to a server (or servers) that a client is still connected to the vehicle and that certain diagnostic services and/or communication that have been previously activated are to remain active. 
    This service is used to keep one or multiple servers in a diagnostic session other than the defaultSession. This can either be done by transmitting the TesterPresent request message periodically or in case of the absence of other diagnostic services to prevent the server(s) from automatically returning to the defaultSession.


    * **Request frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID	|0x3E|
    |2	|zeroSubFunction	|0x00/0x80|

    * **Response frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID	|0x3E|
    |2	|zeroSubFunction	|0x00|



    ## Routine Control -SID=0x31
    The RoutineControl service is used by the client to execute a defined sequence of steps and obtain any relevant results. E.g.: it require the result of checking pre-condition or request peroforming self test,..In particular in the first case, it might be necessary to switch the server in a specific diagnostic session using the DiagnosticSessionControl service or to unlock the server using the SecurityAccess service prior to using the StartRoutine service.

    * **Request frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID 	|0x31|
    |2	|routineControlType	|0x01, 0x02, 0x03|
    |3	|routineIdentifierbyte#1MSB	|0x00-0xFF|
    |4	|routineIdentifierbyte#2LSB	|0x00-0xFF|
    |5..n  |routineControlOptionRecord |0x00–0xFF|

    routineIdentifier: ID of routine, it is specified by OEM<br />
    routineControlType: 0x01,0x02,0x03 is start/stop/routine result accordingly.<br />
    routineControlOptionRecord: this is optional, routine control may only run or sop when satisfy some precondition, these infor is requested via routineControlOptionRecord.

    * **Response frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID 	|0x31|
    |2	|routineControlType	|0x01, 0x02, 0x03|
    |3	|routineIdentifierbyte#1MSB	|0x00-0xFF|
    |4	|routineIdentifierbyte#2LSB	|0x00-0xFF|
    |5	|routineInfo	|0x00-0xFF|

    routineInfo: OEM specific and provides a mechanism for the vehicle manufacturer  to support generic external test equipment handling of all implemented routines 

    ## Security Access-SID=0x27
    This service is to provide a means to access data and/or diagnostic services, which have restricted access for security, emissions, or safety reasons. Diagnostic services for downloading/uploading 
    routines or data into a server and reading specific memory locations from a server are situations where security access may be required.<br/>  

    Security Access service cannot be accessed in default session, therefore, Diagnostic Session Control-SID=0x10 shall be call before if need.<br/>

    The sequence communication between client/server with Security Access service:<br/>

    &ensp; **Client**-----------------------------**Server**<br/>
    &ensp; Request securitySeed-------------><br/>
    &ensp; <----------------------------------securitySeed<br/>
    &ensp; securityKey------------------------><br/>

    Server will verify securityKey, if valid, service will be unlocked with coressponding security level. If Client request the same security level next time, server will response positively without securitySeed.

    The popular algorithm used to authenize and access security in UDS is Hash-based message authentication code HMAC (Hash algorithm can be HMAC_MD5, HMAC_SHA1, HMAC_SHA256, and HMAC_SHA256). Refer [3] for detail of HMAC algorithm, please note that, securityKey here is not the key of HMAC algorithm, we can understand that securitySeed is message need to be encrypted and securityKey is HMAC output from algorthm. Moreover, HMAC is compared by Sender message (Sever) not Receiver (Client).


    * **Request frame format**

    **Request seed from client to server**:
    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|Request SID 	|0x27|
    |2	|requestSeed	|0x01-0x41|
    |3	|securityAccessDataRecordbyte1|0x00 – 0xFF|
    |n	|securityAccessDataRecordbyten|0x00 – 0xFF|

    requestSeed: is odd number, each value corespondent with different levels of security defined by OEM

    **Send key from client to server**:
    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|Request SID 	|0x27|
    |2	|sendKey	|0x02-0x42|
    |3	|securityKeybyte1|0x00 – 0xFF|
    |n	|securityKeybyten|0x00 – 0xFF|

    sendKey: is even number (= corespondent requestSeed + 1), each value corespondent with different levels of security defined by OEM

    * **Response frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|Request SID 	|0x67|
    |2	|securityAccessType	|0x00-0x7F|
    |3	|securitySeedbyte1|0x00 – 0xFF|
    |n	|securitySeedbyten|0x00 – 0xFF|

    securitySeed: this parameter does not need to be in response if server already unlock.


    ## Request Download-SID=0x34
    The requestDownload service is used by the client to initiate a data transfer from the client to the server

    * **Request frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|    
    |1	|SID 	|0x34|
    |2	|DataFormatIdentifier	|0x00-0xFF|
    |3	|AddressAndLengthFormatIdentifier	|0x00~0xFF|
    |4	|MemoryAddress Byte #1	|0x00-0xFF|
    |n	|MemoryAddress Byte #n	|0x00-0xFF|
    |n+1	|MemorySize Byte #1	|0x00-0xFF|
    |n+m	|MemorySize Byte #m	|0x00-0xFF|


    * **Response frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|   
    |1	|SID 	|0x74|	
    |2	|LengthformatIdentifier	|0x00-0xF0|	
    |3	|maxNumberOfBlockLength Byte #1	|0x00-0xFF|
    |n	|maxNumberOfBlockLength Byte #n	|0x00-0xFF|

    ## Transfer Data-SID=0x36

    * **Request frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID 	|0x36|
    |2	|blockSequenceCounter	|0x00-0xFF|	
    |3	|transferRequestParameterRecord#1	|0x00-0xFF|	
    |n	|transferRequestParameterRecord#n	|0x00-0xFF|

    * **Response frame format**

    |No. of bytes   |Description    |Byte value|
    |:--------------|:--------------|:---------|
    |1	|SID 	|0x76|
    |2	|blockSequenceCounter	|0x00-0xFF|


    ## REFERENCE
    [1] ISO14229-1 <br/>
    [2] https://piembsystech.com/read-dtc-information-service-0x19-uds-protocol<br/>
    [3] https://www.baeldung.com/cs/hash-vs-mac#:~:text=MAC%2C%20in%20turn%2C%20is%20an,as%20part%20of%20its%20algorithm<br/>