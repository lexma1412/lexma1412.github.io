---
layout: default
title:  "UDS In AUTOSAR"
---

# UDS In AUTOSAR
In this post, we focus on how manage DTC of ISO14229-1 in Autosar <br />
In AUTOSAR, **DTC, Snapshot data, Extended data are maintained by Dem module**
![Dem Overview](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/Dem_Overview.png?raw=true)

## DTC in AUTSAR
"Event" is a generic term used in AUTOSAR, it covers both DTC and DID definition of UDS. DTC mapped to a ‘Diagnostic event’ of the Dem module. The Dem provides the status of DTC to the Dcm module.

* Dem module supports multiple DTC fomat via (verR20-11) parameter **DemTypeOfDTCSupported**: ISO-14229-1, SAE J2012 OBD DTC, SAE J1939-73, ISO 11992-4, SAE J2012 WWH-OBD DTC. We only mention about ISO-14229-1, so here DTC is 3 bytes format.
![DTC Byte Order](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/Dem_StatusByte.png?raw=true)

* Dem module supports DTC severity according to ISO 14229-1 via parameter **DemDTCSeverity** in containter **DemDTC**
![DemDTCSeverity](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/Dem_Severity.png?raw=true)

## Diagnostic event status managment
As mentioned, each DTC is mapped with Diagnostic event(s)

* The ‘event memory’ is defined as a set of event records located in a dedicated memory block. The event record includes at least the UDS/event status and the event related data. AUTOSAR Dem use **Dem_UdsStatusByteType** to prepresent UDS status byte of ISO-14229-1 for event. SWCs or BSW moudules frequently report monitor results of DTC via the API **Dem_SetEventStatus**. A call of these functions will trigger the status bit processing of the Dem.
![Dem event memory](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/Dem_SetEventStatus.png?raw=true)

* To retrieve UDS status by SW-Cs or other BSW modules use **Dem_GetEventUdsStatus**, Dcm use **GetStatusOfDTC** instead.

* For UDS Status bit transitions, Autosar show conditon to set and reset status bit as below, but it is still compatible with ISO-14229-1 but with further specific constraint and setting:
![Status bit 0](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit0.png?raw=true)
![Status bit 1](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit1.png?raw=true)
![Status bit 2](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit2.png?raw=true)
![Status bit 3](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit3.png?raw=true)
![Status bit 4](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit4.png?raw=true)
![Status bit 5](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit5.png?raw=true)
![Status bit 6](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit6.png?raw=true)
![Status bit 7](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/StatusBit7.png?raw=true)

* Notification of status bit changes: TBD

## Debouncing of diagnostic events

ECU may implement several types of debouncing improving signal quality. Debouncing can be handled by Dem-internally or by SWCs or by other BSW module ,also called monitor in this context, for a specific event. If Debouncing is handled by Dem, according to Autosar, it provide 3 algorithms: Counter based debounce and Timer base debounce and Monitor internal debounce.

### Counter based debounce
A monitor (SWCs or BSW) must trigger the Dem actively by invoking Dem_SetEventStatus() with “DEM_EVENT_STATUS_PREFAILED” or “DEM_EVENT_STATUS_PREPASSED”, usually multiple times, before an event will be qualified as "DEM_EVENT_STATUS_PASSED" or “DEM_EVENT_STATUS_FAILED” whencounter reaches the defined pass/fail threshold. Each separate trigger will add (or subtract) a configured step  size value to a counter value, and the event will be qualified as ‘failed’ or ‘passed’ once this de-bounce counter reaches the respective configured threshold value.<br \>
Parameter **DemDebounceCounterFailedThreshold** used to define the event-specific limit indicating the failed status (active).
Parameter **DebounceCounterPassedThreshold** used to define the event-specific limit indicating the passed status (passive).
Parameter **DemDebounceCounterIncrementStepSize** used to define the increasing step-szie when the monitor reports DEM_EVENT_STATUS_PREFAILED.
![Example of counter based debouncing](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/Counterdebounce.png?raw=true)

### Timer base debounce
The Dem module shall start the internal debounce timer to qualify the reported event as failed when the monitor reports DEM_EVENT_STATUS_PREFAILED
![Example of Time based debouncing](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/UDS_in_AUTOSAR/Timedebounce.png?raw=true)

### Monitor internal debounce
This setting, Dem module shall not use a Dem internal debounce mechanism for each individual event, to qualify the reported event. If monitor internal de-bouncing is configured for an event, its monitor cannot request debouncing by the Dem (i.e. trigger operation SetEventStatus with monitor results DEM_STATUS_PRE_FAILED or DEM_STATUS_PRE_PASSED). we can understand that monitor internal is implemented by monitor/software component.


## Event related data
In Dem Autosar, : snapshot data (freeze frames) and extended data are called related data

### REFERENCE
[1] Autosar Diagnostic Event Manager R22-11
