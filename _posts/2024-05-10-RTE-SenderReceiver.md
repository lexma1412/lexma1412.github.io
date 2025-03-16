# Sender-Receiver in RTE
This post investigate how Sender-Receiver communication between SWCs are implemented according to AUTOSAR RTE standard.
Sender-receiver communication involves the transmission and reception of signals consisting of atomic data elements that are sent by one component and received by one or more components. A sender-receiver interface can contain multiple data elements.

## 1. Receive Modes
There are 4 receive modes for Sender-Receiver:
* Implicit data read access
* Explicit data read access
* Wake up of wait point
* Activation of runnable entity

Above modes can be combined together, below is rule:



## 1.1 Implicit data read access
RTE makes a "copy" of the data and runnable have access to a "copy" of the data that remains unchanged during the execution
of the runnable.<br/>
```
SWC:
    COPY_A // copy of DATA_A
    IM_DATA_A // immediate DATA_A
    DATA_A // actual DATA_A

    Runnable_1():
        Rte_IRead_COPY_A(&copy_A) ; copy_A = COPY_A
        .. do something with copy_A

    Runnable_2():
       .. do something
       Rte_IWRITE_IM_DATA_A_A(data_1) ; Set IM_DATA_A = data_1
       .. do something IM_DATA_A
       Rte_IWRITE_IM_DATA_A_A(data_2) ; Set IM_DATA_A = data_2

Task_10ms()
{
    COPY_A = DATA_A; Read a copy data of A before invoking Runnable_1
    Runnable_1() ; Call Runnable_1
}

Task_5ms()
{
    Runnable_2() ; Call Runnable_2
    DATA_A = IM_DATA_A ; Write actual Data A only once
}
```

A VariableAccess in the dataReadAccess role references the VariableDataPrototype.<br/>

### Implicit Communication Behavior in case of incoherent implicit data access



## 1.2 Explicit data read access
The RTE generator creates a **blocking or non-blocking** API call to enable a receiver to poll (and read) data. This receive mode is an "explicit" mode since an explicit API call is invoked by the receiver.<br/>
-If the call is non-blocking (i.e. there is a VariableAccess in the dataReceivePointByValue or dataReceivePointByArgument role referencing the VariableDataPrototype for which the API is being generated, but no WaitPoint referencing a DataReceivedEvent
which references the VariableDataPrototype for which the API is being generated), the API call immediately returns the next value to be read and, if the communication is queued (event reception), it removes the data from the receiver-side queue.<br/>
-In contrast, a blocking call (i.e. the VariableDataPrototype, referenced by a VariableAccess in the role dataReceivePointByArgument, and for which the API is being generated, is referenced by a DataReceivedEvent which is itself referenced by a WaitPoint) will suspend execution of the caller until new data arrives (or a timeout occurs) at the according port. When new data is received, the RTE resumes the execution of the waiting runnable.<br/>


A VariableAccess in the dataReceivePointByValue or dataReceivePointByArgument role references the VariableDataPrototype.<br/>


## 1.3 Wake up of wait point
The RTE generator creates a blocking API call that the receiver invokes to read data<br/>
The “wake up of wait point” receive mode shall support a **time-out** to prevent infinite blocking if no data is available.


## 1.4 Activation of runnable entity
The receiving runnable entity is invoked automatically by the RTE whenever new data is available. To access the new data,
the runnable entity either has to use “implicit data read access” or “explicit data
read access”, i.e. invoke an Rte_IRead, Rte_Read, Rte_DRead or Rte_Receive call, depending on the input configuration. This receive mode differs from “implicit data read access” since the receiver is invoked by the RTE in response to a DataReceivedEvent.


# 2. Multiple Data Elements