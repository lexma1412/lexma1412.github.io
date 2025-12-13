---
layout: default
title:  "UDS Data Parameter"
---

# UDS DATA PARAMETER

In this post, we will go into detail about each kind of data parameter supported in UDS and also link it with AUTOSAR standards.

## DTC

First, we need to clarify some definition<br/>
**DTC status**: below is the definition of DTC status from ISO14229-1; each DTC has 1 byte to contain 7 bits with varying meanings:

|Bit            |DTC status bit |Description    |
|:--------------|:--------------|:--------------|
|0| testFailed | indicate the result of the most recently performed test<br />"1": means the most recent result from DTC test is fail|
|1| testFailedThisOperationCycle        | indicate whether or not a diagnostic test has reported a testFailed result at any time during the current operation cycle. Reset to logical '0' when a new operation cycle is initiated or after a call to ClearDiagnosticInformation. |
|2| pendingDTC                          | indicate whether or not a diagnostic test has reported a testFailed result at any time during the current or last completed operation cycle. The status shall only be updated if the test runs and completes. pendingDTC bit is not cleared until an operation cycle has completed where the test has passed at least once and never failed|
|3| confirmedDTC | indicate whether a malfunction was detected enough times to warrant that the DTC is desired to be stored in long-term memory. confirmedDTC does not always indicate that the malfunction is present at the time of the request. Reset to logical '0' after a call to ClearDiagnosticInformation or after aging threshold has been satisfied. DTC confirmation threshold and aging threshold are defined by the vehicle manufacturer or mandated by On Board Diagnostic regulations.|
|4| testNotCompletedSinceLastClear      | indicate whether a DTC test has ever run and completed since the last time a call was made to ClearDiagnosticInformation.|
|5| testFailedSinceLastClear            | indicate whether a DTC test has completed with a failed result since the last time a call was made to ClearDiagnosticInformation|
|6| testNotCompletedThisOperationCycle  | indicate whether a DTC test has ever run and completed during the current operation cycle|
|7| warningIndicatorRequested           | report the status of any warning indicators associated with a particular DTC. Warning outputs may consist of indicator lamp(s), displayed text information, etc.|


**Severity and class definition** of DTC from ISO14229-1.
The DTCSeverityMask / DTCSeverity byte contains DTC severity and DTC class information. The DTCSeverityMask / DTCSeverity byte is reported in a 1-byte value as defined. The optional upper 3 bits (bits 7-5) of the 1-byte value are used to represent the DTC severity information. If not supported by the server, those bits shall be set to "0". The mandatory lower 5 bits (bits 4-0) of the 1-byte value are used to represent the DTC class information. However, in a real project, there may be cases where class definition is ignored because the system does not need to comply with the WWH-OBD GTR Class A, B1, B2, or C (bits 0-4 are 0).


|Bit            |Description    |
|:--------------|:--------------|
|5|**maintenanceOnly**<br />0 = no maintenanceOnly severity<br />1 = maintenanceOnly severity<br /> This value indicates that the failure requests maintenance only.|
|6|**checkAtNextHalt**<br />0 = do not checkAtNextHalt<br />1 = checkAtNextHalt <br />This value indicates to the failure that a check of the vehicle is required at next halt.|
|7|**checkImmediately**<br />0 = do not checkImmediately<br />1 = checkImmediately<br />This value indicates to the failure that an immediate check of the vehicle is required.|


## UDS EXTENDED DATA

ExtendedData consists of extended status information associated with a DTC; the extended data varies from project to project. This post introduces some extended data that might be helpful.

### Occurrence Counter
This counter indicates how often errors happen. In simple terms, it counts the number of cycles which has test failures. The formal condition to count up the occurrence counter is when testFailedThisOperationCycle changed from "0" to "1".

### Aging Counter
- Counts *driving cycles with no fault* after a DTC failure.
- Increments only when:
  - testPassed reported, AND
  - testFailed NOT reported in that cycle.
- Resets to 0 if a new testFailed occurs.
- When it reaches AgingTarget, the DTC is considered “Aged”.

**Think of it as a progress bar toward healing.**

### Example:
AgingTarget = 3  
Failure → AgingCounter = 0  
Cycle 1 (no fail) -> 1  
Cycle 2 (no fail) -> 2  
Cycle 3 (no fail) -> 3 -> Aging reached


### Aged Counter
- Counts *how many times* a DTC has **fully** completed the aging process.
- Increments only when AgingCounter reaches AgingTarget.
- Never resets unless memory is cleared.

**Think of it as: how many times the DTC has fully healed in its lifetime.**

### Example:
1st time DTC aging completed -> AgedCounter = 1  
DTC fails again and ages again -> AgedCounter = 2  


### Fault Detection Counter
This counter is based on debounce strategy, ranging from -128 to 127, to demonstrate prepass/prefail status before the counter reaches the pass/fail threshold to indicate an actual pass/fail.