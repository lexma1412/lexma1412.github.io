---
layout: default
title:  "Mode Management"
---

# Mode Management for Communcation
## Use case: SWC request COM mode directly to ComM
In this use case, SWC that wants to explicitly direct the local Communication Manager Module of the ECU towards a certain state requires the client-server interface ComM_UserRequest. Through this interface the SW-C can set the desired state of all communication channels that are relevant for that component, to “No Communication” or “Full Communication”.

|API/Operation            |Port interface |Port Type      |Description|
|:--------------|:--------------|:--------------|:--------------|
|ComM_RequestComMode|ComM_UserRequest|Client/Server|SWC request mode for ComM|
|ComM_GetCurrentComMode|ComM_UserRequest|Client/Server|Returns the current Communication Manager Module mode|

Another approach to get current ComM mode instead of ComM_GetCurrentComMode operation (of Client/Server port) is using ModeSwitch port **ComM_CurrentMode**. This approach can be used in case SWC only want to current get COM mode.

## Use case: SWC request channel mode to BswM, BswM to CanSM
In this use case, SWC request mode to BswM through configuration choice container **BswMModeRequestPort/BswMModeRequestPort**, in case mode requester is SWC, BswMSwcModeRequest container is chosen (The source of the mode request is a SW Component).

Base on configured rulelist, BswM triggers Autosar action C-API **BswMComMModeSwitch** (it is a container of BswM) that will call ComM_RequestComMode to request mode to coressponding CanSM channel. In general, BswM can trigger Autosar action C-API or PPort of Client/Sever or ModeSwitch depend on design.

## Use case: SWC request inhibit channel mode to BswM, BswM to ComM
In this use case, SWC request mode to BswM through configuration choice container **BswMModeRequestPort/BswMModeRequestPort**, in case mode requester is SWC, BswMSwcModeRequest container is chosen (The source of the mode request is a SW Component).

Base on configured rulelist, BswM triggers AUtosar RPORT **ComM_ChannelLimitation** client/server interface to configure the Communication Manager Module to inhibit communication mode for a given channel. In other words, SWC and BswM does control ComM mode fully, it just can do the "inhibit".

|API/Operation            |Port interface |Port Type      |Description|
|:--------------|:--------------|:--------------|:--------------|
|ComM_GetInhibitionStatus|ComM_ChannelLimitation|Client/Server|returns the inhibition status of a channel|
|LimitChannelToNoComMode|ComM_ChannelLimitation|Client/Server|Changes the inhibition status for the channel for changing from COMM_NO_COMMUNICATION to a higher Communication Mode.|

## Use case: ComM or CanSM or CanNM indicate mode change to BswM

|API/Operation            |Port interface |Port Type      |Description|
|:--------------|:--------------|:--------------|:--------------|
|BswM_CanSM_CurrentState|BswMCanSMIndication|ModeRequestPort/BswMModeRequestPort|choice container|This is an indication of the current state of the CANSM|
|BswMComMIndication|BswMCanSMIndication|ModeRequestPort/BswMModeRequestPort|choice container|This is an indication of the current communication mode of a channel in the ComM.|
|BswMNmStateChangeNotification|BswMCanSMIndication|ModeRequestPort/BswMModeRequestPort|choice container|This is a notification from the Nm module that its state has changed|

Each of above choice container has reference parameter to indicate coressponding channel requesting mode.

The action of BswM: TBD

## Use case: SWC request channel mode directly to ComM
In Autosar, there is no recommendation to support this use case. Instead, SWC shall request communication channel mode through BswM.

# Mode Management for SWC
Mode will be managed via RTE in this case. There are 2 approach to manage mode is Sender/Receiver and ModeSwitch Port. With ModeSwitch Port, it provide ModeSwitchEvent and  RTEEvent to trigger or disable Execuable Entity (e.g: trigger )

## Use case: SWC request mode to BswM, BswM broadcast to other SWCs
In this use case, SWC request mode to BswM through configuration choice container **BswMModeRequestPort/BswMModeRequestPort**, in case mode requester is SWC, BswMSwcModeRequest container is chosen (The source of the mode request is a SW Component).
After performing action to change mode, BswM indicate mode to SWCs via ModeSwitch port **BswM_modeSwitchPort**

## Use case: SWC send mode to other SWCs
In this use case, SWC handling mode control called mode manager. Sender/Receiver PPORT is created for each SWCs to request mode to mode manager SWC and ModeSwitch PPORT of mode manager SWC to indcate mode to all mode user SWCs.<br/>

<br/>**Todo**:<br/>
-Add picture for post<br/>
# Reference
1. Guide to Mode Management
2. BswM
3. ComM
