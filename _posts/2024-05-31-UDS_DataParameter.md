---
layout: default
title:  "UDS Data Parameter"
---

# UDS DATA PARAMETER

In this post, we will go in detail each kind of Data parameter supporteed in UDS and also link it with AUTOSAR standard.

## DTC

First, we need to clarify some definition<br/>
**DTC status**: below is definition of DTC status from ISO14229-1, each DTC has 1byte to contain 7 bit with vary meanning:

|Bit            |DTC status bit |Description    |
|:--------------|:--------------|:--------------|
|0| testFailed                          | indicate the result of the most recently performed test<br />"1": mean most recent result from DTC test is fail|
|1| testFailedThisOperationCycle        | indicate whether or not a diagnostic test has reported a testFailed result at any time during the current operation cycle. Reset to logical '0' when a new operation cycle is initiated or after a call to ClearDiagnosticInformation. |
|2| pendingDTC                          | indicate whether or not a diagnostic test has reported a testFailed result at any time during the current or last completed operation cycle. The status shall only be updated if the test runs and completes. pendingDTC bit is not cleared until an operation cycle has completed where the test has passed at least once and never failed|
|3| confirmedDTC                        | indicate whether a malfunction was detected enough times to warrant that the DTC is desired to be stored in long-term memory.  confirmedDTC does not always indicate that the malfunction is present at the time of the request.Reset to logical '0' after a call to ClearDiagnosticInformation or after aging threshold has been satisfied. DTC confirmation threshold and aging threshold are defined by the vehicle manufacturer or mandated by On Board Diagnostic regulations.|
|4| testNotCompletedSinceLastClear      | indicate whether a DTC test has ever run and completed since the last time a call was made to ClearDiagnosticInformation.|
|5| testFailedSinceLastClear            | indicate whether a DTC test has completed with a failed result since the last time a call was made to ClearDiagnosticInformation|
|6| testNotCompletedThisOperationCycle  | indicate whether a DTC test has ever run and completed during the current operation cycle|
|7| warningIndicatorRequested           | report the status of any warning indicators associated with a particular DTC. Warning outputs may consist of indicator lamp(s), displayed text information, etc.|


**Severity and class definition** of DTC from ISO14229-1.
The DTCSeverityMask / DTCSeverity byte contains DTC severity and DTC class information. The DTCSeverityMask / DTCSeverity byte is reported in a 1-byte value as defined . The optional upper 3 bits (bit 7-5) of the 1-byte value are used to represent the DTC severity information. If not supported by the server those bits shall be set to "0". The mandatory lower 5 bits (bit 4-0) of the 1-byte value are used to represent the DTC class information. However in real project, there is case that class definition is ignored because system does not need to comply with with the WWH-OBD GTR Class A, B1, B2 or C (bit 0-4 are 0).


|Bit            |Description    |
|:--------------|:--------------|
|5|**maintenanceOnly**<br />0 = no maintenanceOnly severity<br />1 = maintenanceOnly severity<br /> This value indicates that the failure requests maintenance only.|
|6|**checkAtNextHalt**<br />0 = do not checkAtNextHalt<br />1 = checkAtNextHalt <br />This value indicates to the failure that a check of the vehicle is required at next halt.|
|7|**checkImmediately**<br />0 = do not checkImmediately<br />1 = checkImmediately<br />This value indicates to the failure that an immediate check of the vehicle is required.|


## UDS EXTENDED DATA

ExtendedData consists of extended status information associated with a DTC, the extended data is specified vary from project. This post introduce some extended data that might helpful.

### Occurrence Counter
This counter indicate how often error happen, in easy way, it counts number of cycles which has test fail. The formal condition to count up occurrence counter is testFailedThisOperationCycle changed from "0" to "1".

### Aging Counter
Aging Counter counts number of driving cycles since the fault was latest failed excluding the driving cycles in which the test has not reported "testPassed" or "testFailed".

### Aged Counter


### Fault Detection Counter
This counter is based on debounce strategy, range from -128..127 to demonstrate prepass/prefail status before counter reachs the pass/fail threshold to indicate it is actual pass/fail.