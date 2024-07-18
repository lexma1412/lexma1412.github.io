---
layout: default
title:  "PduR"
---

# PDU Router
The PDU Router module is an I-PDU transfer unit from upper to lower layer, it rout/foward I-PDU from upper layer to lower layer and vice versa. PDU Router can also handle the role of gateway between If modules or Tp modules.

More detail, from the ECU point of view, the PDU Router module can perform three different classes of operations:
* **PDU Reception** to local module(s): receive I-PDUs from lower layer modules
and forward them to one or more upper layer modules,
* **PDU Transmission** from local module(s): transmit I-PDUs to one or more lower
layer modules on request of upper layer module,
* **PDU Gateway:**</br>
&nbsp;Receive I-PDUs from a Communication Interface module and transmit the
I-PDUs immediately via the same or other Communication Interface module(s) -If.</br>
&nbsp;receive I-PDUs from a Transport Protocol module and transmit the I-PDUs
via the same or other Transport Protocol module(s)-TP.</br>

The Routed I-PDU can come from user/upper layer: Dcm, COM, IpduM... and sent to lower layer: TP, If (CanTp, FrTp, FrIf, CanIf, LinIf). For Ethernet, it is more complicated, so we may discuss Eth in other post.</br>

The routing relationship between the number upper layer (of PduR) and lower layer (of PduR) can be Singalcast(1:1), Multicast(1:N) or Fan-in(N:1). <br/>

The capability of forward (PDU Reception/Tranmission) feature is below ([1]):

|Direction  |Routing method |Capability     |
|:----------|:--------------|:--------------|
|Tx         |1:1            |Capable        |
|           |1:N            |Capable        |
|Rx         |1:1            |Capable        |
|           |1:N            |Capable, need buffer FIFO in case Tp|
|           |N:1            |Capable, need buffer in case If|

To use Gateway feature, PduR need to implement buffer mechanism. The capability of gateway feature is below ([1]):

|Routing method |Buffer         |Capability     |
|:--------------|:--------------|:--------------|
|1:1            |Last is Best	|Capable        |
|               |FIFO           |Capable        |
|               |No Buffer	    |Not Capable    |
|1:N            |Last is Best	|Capable        |
|               |FIFO           |Capable        |
|               |No Buffer	    |Not Capable    |
|N:1            |Last is Best	|Only 1 source enabled at a time (1:1)|
|               |FIFO           |Only 1 source enabled at a time (1:1)|
|               |No Buffer	    |Not Capable    |


# I-PDU reception

##  Communication Interface (<Module>If)
<Module>If Lower layer indicates a received I-PDU to PduR by calling **PduR_<User:Lo>RxIndication** (e.g: PduR_CanTpRxIndication, PduR_CanIfRxIndication). After that PduR forward to upper layer(s) by calling **<Up>_RxIndication** (e.g: Com_RxIndication, Dcm_RxIndication)

## Transport Protocol (<Module>Tp)

In case Transport Protocol, I-PDU is segmented into N-PDU(s), therefore, need to perform sequence to copy consecutive N-PDU(s) into I-PDU of upper layer. 

Below is sequence for reception 1:1:
|Step           |Description    |
|:--------------|:--------------|
|1 |Indicate start reception to PduR PduR_<User:LoTp>StartOfReception|
|2 |Indicate start reception to uppder layer <Up>_StartOfReception|
|3 |Request PduR to copy data PduR_<User:LoTp>CopyRxData, PduR only fowrard data by step 4|
|4 |Request upper layer to copy data into PDU <Up>_CopyRxData|
|5 |Indicate complete reception to PduR PduR_<User:LoTp>RxIndication|
|6 |Indicate complete reception to upper layer <Up>_TpRxIndication|

Below is sequence for reception 1:N (optional support), data from lower layer shall be received completedly in buffer before forwarding to upper layer:

|Step           |Description    |
|:--------------|:--------------|
|1 |Indicate start reception to PduR PduR_<User:LoTp>StartOfReception|
|2 |Request PduR to copy data -> buffer data|
|3 |Indicate complete reception to PduR PduR_<User:LoTp>RxIndication|
|4 |Indicate start reception to uppder layers <Up>_StartOfReception|
|5 |Request upper layers to copy data into PDU <Up>_CopyRxData|
|6 |Indicate complete reception to upper layers <Up>_TpRxIndication|

# I-PDU transmission
The transmit operation of the PDU Router module is triggered by a PDU Transmit
request from an upper layer source module and the PDU Router forwards the request
to lower layer destination(s).
If the PDU Router module is notified by lower layer destination modules via PduR_<User:Lo>TxConfirmation (Communication Interface) or PduR_<User:LoTp>TxConfirmation (Transport Protocol) after successful or failed transmission of the I-PDU, the PDU Router module will forward this confirmation to the upper layer module via <Up>_TxConfirmation (Communication Interface) or <Up>_TpTxConfirmation (Transport Protocol).

##  Communication Interface (<Module>If)

There are 3 ways to tranmission to If:

* **Direct data provision** - where the upper layer module is calling the PduR_<User:Up>Transmit function, the PDU Router module forwards the call to <Lo>_Transmit and the data is copied by the lower Communication Interface module in the call.

* **Trigger transmit provision** - where the lower Communication Interface module requests transmission of an I-PDU by using the PduR_<User:Lo>TriggerTransmit, and PDU Router module forwards the call to <Up>_TriggerTransmit and the data is copied to the destination’s buffer by the upper layer module. This is useful if the destination module always wants to fetch the latest data.

* Where the upper layer module calls the PduR_<User:Up>Transmit function, the PDU Router module forwards the call to <Lo>_Transmit and the data is not copied by the lower module (Communication Interface module). The data will later be requested by the lower layer using PduR_<User:Lo>TriggerTransmit.

When the Communication Interface module calls PduR_<User:Lo>TxConfirmation the PDU Router shall call <Up>_TxConfirmation in the upper layer module and forward the transmission result from the lower to the upper layer module

## Transport Protocol (<Module>Tp)

Below is sequence for transmission 1:1:
|Step           |Description    |   
|:--------------|:--------------|
|1 |Upper layer request transmission to PduR PduR_<User:Up>Transmit|
|2 |PduR forward request to lower Tp <Lo>_Transmit|
|3 |Lower Tp request data by calling the PduR_<User:LoTp>CopyTxData|
|4 |PduR forward request data to upper layer <Up>_TpTxConfirmation|
|5 |Lower Tp confirm the transmission to PduR PduR_<User:LoTp>TxConfirmation|
|6 |PduR confirm the transmissio to upper layer <Up>_TxConfirmation|

For transmission 1:N the sequence is almost same 1:1, but with some additional requirement:
- For 1st time request data, TpDataState = TP_CONFPENDING
- For subsequential time request data,transport protocol destinations DO NOT set the TpDataState to TP_DATARETRY, else PduR shall return E_NOT_OK.
- For subsequential time request data, PduR shall overwrite TpDataState to TP_DATARETRY and adjust Tx TpData Cnt to request data to be transmitted from the upper layer source buffer from the previous transmission point related to that destination
- PDU Router module shall call the upper layer module using \<Up\>\_TpTxConfirmation after receiving the last PduR\_\<User\:LoTp\>TxConfirmation from the lower layer Transport Protocol modules. The result parameter shall be E_OK if at least one PduR\_\<User\:LoTp\>TxConfirmation reported E_OK

# I-PDU gateway

##  Communication Interface (<Module>If)
I-PDUs may be gatewayed 1:1, 1:n or n:1
- PDU Router module may set the type of buffering for each destination independently (FIFO, single buffer (last is best))
- An I-PDU may be received by destinations of upper layer module(s) at the same time as gatewayed to n destinations of Communication Interface. (forward + gateway)

* Direct data provision: The PduRDestPduDataProvision of the destination I-PDU is configured to PDUR_DIRECT. When <DstLo>_Transmit is called the <DstLo> module copies the data and the PDU Router does not buffer the transmitted I-PDU any longer.

* Trigger transmit data provision: The PduRDestPduDataProvision of the destination I-PDU is configured to PDUR_TRIGGERTRANSMIT. When \<DstLo\>\_Transmit is called the <DstLo> module does not copy the data and the PDU Router module shall buffer the I-PDU and wait for the PduR_<DstLo>TriggerTransmit call from the <DstLo> module

* Buffered gateway:  TBD

## Transport Protocol (<Module>Tp)
* Direct gatewaying: complete set of N-PDUs building up the I-PDU is received before
transmitted
* On-the-fly gatewaying: I-PDUs where a configured number of bytes (PduRTpThreshold) are received before transmission

# Reference
[1] https://www.autosartoday.com/posts/demystifying_autosar_pdu_router_a_comprehensive_guide_to_routing_handling_and_buffering