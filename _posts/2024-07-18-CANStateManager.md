---
layout: default
title:  "CAN State Manager"
---

This post summarize CanSM and also try to explain how Can 
The CanSM module is responsible for the control flow abstraction of CAN networks:
It changes the communication modes of the configured CAN networks depending on the **mode requests from the ComM module**. Therefore the CanSM module uses the API of the CanIf module. The CanIf module is responsible for the control flow abstraction of the configured CAN Controllers and CAN Transceivers.

Any change of the CAN Controller modes and CAN Transceiver modes will be notified by the CanIf module to the CanSM module. Depending on this notifications and state of the CAN network state machine, which the CanSM module shall implement for each configured CAN network, the CanSM module notifies the ComM and the BswM 

## Request Mode from ComM to CanSM
ComM invokes CanSM_RequestComMode API provided by CanSM
```
Std_ReturnType CanSM_RequestComMode(
NetworkHandleType network,
ComM_ModeType ComM_Mode
)
```
ComM_Mode parameter can be set to 1 of 3 values:
|Value            |Description  |
|:--------------|:--------------|
|COMM_NO_COMMUNICATION| ComM state machine is in "No Communication" mode. Configured channel shall have no transmission or reception capability.|
|COMM_SILENT_COMMUNICATION| ComM state machine is in "Silent Communication" mode. Configured channel shall have only reception capability, no transmission capability.|
|COMM_FULL_COMMUNICATION| ComM state machine is in "Full Communication" mode. Configured channel shall have both transmission and reception capability.|

When receiving the request mode, CanSM performed CanSM state change internally, internal state machine of CanSM is complecated (TBD). After that, CanSM inform the current network state of CanSM(acually that is state from Can controller ) to ComM by invoking ComM API **ComM_BusSM_ModeIndication(Channel, ComMode)** cyclic in CanSM_Mainfunction. Pesudo code:

```
# ComM request mode to CanSM
Function CanSM_RequestComMode(network, ComM_Mode)
    # Store mode
    CurCanSMMode = ComM_Mode

# CanSM request mode to CanIf
Function CanSM_MainFunction()
    # Asynchorous mode request to CanIf
    CanIf_SetControllerMode(network, CurCanSMMode)

# CanIf inform actual mode to CanSM
Function CanSM_ControllerModeIndication(CanIf_ModeIndication)
    IF CurCanSMMode != CanIf_ModeIndication
        # Inform to upper layer ComM
        ComM_BusSM_ModeIndication(network, CurCanSMMode)
        # Inform to upper layer BswM
        BswM_CanSM_CurrentState(Network, CurrentState)
```

## Request Mode from CanSM to CanIf

To forward network mode from ComM to lower layer CanIf, CanSM invokes
```
CanIf_SetControllerMode(
    ControllerId, 
    ControllerMode
    )
```
ControllerMode can be 1 of 3 values:
|Value          |Description    |
|:--------------|:--------------|
|CAN_CS_STARTED| CAN controller state STARTED. The function Can_SetControllerMode(CAN_CS_STARTED) shall set the hardware registers in a way that makes the CAN controller participating  on the network.|
|CAN_CS_STOPPED| CAN controller state STOPPED. The function Can_SetControllerMode(CAN_CS_STOPPED) shall set the bits inside the CAN hardware such that the CAN controller stops participating on the network.|
|CAN_CS_SLEEP| CAN controller state SLEEP.The function Can_SetControllerMode(CAN_CS_SLEEP) shall set the controller into sleep mode. If the CAN HW does not support a sleep mode, the function Can_SetControllerMode(CAN_CS_SLEEP) shall set the CAN controller to the logical sleep mode|

CanIf notify upper layer CanSM with
```
CanSM_ControllerModeIndication(
    int8 ControllerId,
    Can_ControllerStateType ControllerMode
)
```