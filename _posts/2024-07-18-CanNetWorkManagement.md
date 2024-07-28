---
layout: default
title:  "CAN NetWork Managment"
---

# CAN NetWork Managment
This post introduce how to manage CAN Network according to Autosar NM (R22-11).  Its main purpose is to coordinate the transition between normal operation and bus-sleep mode of the network (no message sent on bus).

# CAN NM overview

The AUTOSAR CanNm Coordination algorithm is based on periodic Network Management message, every ECU node has responsibility to periodcally transmit network management message which used for network manangement, as long as there is NM message, any ECU node(s) want to transit to Bus Sleep mode need to be postponed because it still received NM message on bus.

The main concept of the AUTOSAR CanNm algorithm can be defined by the 
following two key-requirements:<br/>
- Every network node in a CanNm cluster shall transmit periodic Network Management PDUs as long as it requires bus-communication; otherwise it shall not transmit NM message.

- If CanNmStayInPbsEnabled is disabled and bus communication in a CanNm cluster is released and there are no Network Management PDUs on the bus for a configurable amount of time determined by CanNmTimeoutTime + CanNmWaitBusSleepTime (both configuration parameters) transition into the Bus-Sleep Mode shall be performed.<br/>

For single node point of view, Can NM state machine consists of three internal <span style="color:red">**(operation) modes**</span> (In CanNM context, mode contains multiple states): Network Mode, Prepare Bus-Sleep Mode, Bus-Sleep Mode.<br/>

CanNM also provide 2 addition state called <span style="color:red">**Network states**</span> that exist in parallel with operation modes:
* Request: need to communicate on the bus
* Release: don’t  have to communicate on the bus (the bus network state is then ‘released’); note that if the network is released an ECU may still communicate because some other ECU 
still request the network.

##  Operational Modes

![Can NM mode](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/CanNM/CanNM_Mode.png?raw=true)

### Network Mode
Network Mode consists of 3 internal states:
* **Repeat Message State**: ensures any transition from Bus-Sleep or Prepare Bus-Sleep to the Network Mode becomes visible to the other nodes on the network. Additionally, it ensures that any node stays active for a minimum amount of time.
* **Normal Operation State**: ensures that any node can keep the network 
management cluster awake as long as the network is requested.
* **Ready Sleep State**: ensures that any node in the network management cluster
waits with transition to the Prepare Bus-Sleep Mode as long as any other node keeps 
the network management cluster awake.

### Prepare Bus-Sleep Mode
Prepare Bus-Sleep Mode ensures that all nodes have time to 
stop their network activity before the Bus-Sleep Mode is entered. In Prepare Bus Sleep Mode the bus activity is calmed down (i.e. queued messages are transmitted in order to make all Tx-buffers empty) and finally there is no activity on the bus in the Prepare Bus-Sleep Mode.

### Bus-Sleep Mode
Bus-Sleep Mode reduce spower consumption in the node when no messages are to be exchanged. The communication controller is switched into the sleep mode, respective wakeup mechanisms are activated and finally power consumption is reduced to the adequate level in the Bus-Sleep Mode.<br/>

To indicate upper layer (NM) about current state of Can NM, CAN NM call callback function of NM, after that NM forward this indication to ComM via callback ComM_Nm_NetworkMode, ComM_Nm_BusSleepMode and ComM_Nm_PrepareBusSleepMode:
|||
|:--------------|:--------------|
|NM_BusSleepMode|Notification that the network management has entered Bus-Sleep Mode.|
|NM_NetworkMode| Notification that the network management has entered Network Mode.|
|Nm_PrepareBusSleepMode|Notification that the network management has entered Prepare BusSleep Mode (or it means Leaving Network Mode).|

In bus-sleep mode, if node recevied CanNM message on bus, CanNM will indicate to NM by calling calback **Nm_NetworkStartIndication**, NM will forward to ComM by calling **ComM_Nm_NetworkStartIndication**. In ComM, it will judge about the network wakeup and request to NM to CanNM via **Nm_PassiveStartup/CanNm_PassiveStartup** to make a transition modoe in CanNM.

## NM Message
To keep Network still awake, NM is sent periodcally on bus, the structure of NM message:

![Can NM message](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/CanNM/CanNM_Message.png?raw=true)

### Control bit Vector (CBV)

![Can NM Control bit Vector](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/CanNM/CanNM_ControlbitVector.png?raw=true)

### User data

## User data configuration
The usage of user data is not standardized and specified by OEM.<br/>
To able to update user data in NM message, parameter **CanNmUserDataEnabled** = True and invoke API **CanNm_SetUserData** from SWC.<br/>
CaNM request the transmission of NM message to CanIf module via **CanIf_Transmit()**, ater that, CanIf confirm to CanNM the status of transmission via **CanNm_TxConfirmation()**


## Com User data configuration
Alternatively to the usage of the CanNm APIs to set and get user data, CanNm may 
use the COM to retrieve its user data by enabling ****CanNmComUserDataSupport**. In this case **CanNm_SetUserData** shall not be available.<br/>
To request the transmit of new user data, CanIf call **CanNm_TriggerTransmit(PduIdType TxPduId, PduInfoType\* PduInfoPtr)**, CanNM shall collect the NM User Data from the referenced NM I-PDU by calling ****PduR_CanNmTriggerTransmit**.<br/>. After that, CanIf confirm by calling **CanNm_TxConfirmation** and CanNM call **PduR_CanNmTxConfirmation**.
To get most recently NM message, **CanNm_GetUserData** is invoked.<br/>

## Passive mode
In the Passive Mode the node is only receiving Network Management PDUs but not 
transmitting any Network Management PDUs. To enable, set **CanNmPassiveModeEnabled = True **<br/>
Therefore, in passive mode, Normal Operation State/Ready Sleep State has no action (no need to send NM message).

## State change notification
All changes of the AUTOSAR CanNm states shall be notified to the upper layer by calling callback function **Nm_StateChangeNotification** if parameter **CanNmStateChangeIndEnabled** is enabled. This way is optional.<br/>
As mentioned above, main approach for each enter mode can notified via callback ComM_Nm_NetworkMode, ComM_Nm_BusSleepMode and ComM_Nm_PrepareBusSleepMode.

## Tranmission NM
The tranmission NM message only possible if Passive Mode is disable (CanNmPassiveModeEnabled = False). For how to tranmission, refer User data configuration and Com User data configuration.

## Reception NM
If a NM PDU has been successfully received, the CanIf module will call the callback 
function CanNm_RxIndication.<br/> 
CanNm module shall call the Nm callback function Nm_PduRxIndication, if and only if 
CanNmPduRxIndicationEnabled (configuration parameter) is set to TRUE


## Bus load Reduction
TBD

## Partial Networking
TBD

## Bus sleep indication, synchronization
The “Remote Sleep Indication” denotes a situation, where a node in Normal
Operations States finds all other nodes in the cluster are ready to sleep (in ReadySeep State). The node in Normal Operation State will still keep the bus awake. To enable, set parameter **CanNmRemoteSleepIndEnabled**<br/>

With a call of Nm_RemoteSleepIndication CanNm notifies the module Nm that all
nodes in the cluster are ready to sleep (the so-called ‘Remote Sleep Indication’)<br/>

With a call of Nm_RemoteSleepCancellation CanNm notifies the module Nm that some
nodes in the cluster are not ready to sleep anymore (the so-called ‘Remote Sleep
Cancellation’).<br/>


## Sequence diagram

![Can NM coordinator](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/CanNM/CanNM_sqNMCoordination.png?raw=true)

