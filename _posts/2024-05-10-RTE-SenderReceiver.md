# Sender-Receiver in RTE
This post investigate how Sender-Receiver communication between SWCs are implemented according to AUTOSAR RTE standard.
Sender-receiver communication involves the transmission and reception of signals consisting of atomic data elements that are sent by one component and received by one or more components. A sender-receiver interface can contain multiple data elements.

## 1. Receive Modes
There are 4 receive modes for Sender-Receiver:
* Implicit data read access
* Explicit data read access
* Wake up of wait point
* Activation of runnable entity


## 1.1 Implicit data read access
RTE makes a "copy" of the data and runnable have access to a "copy" of the data that remains unchanged during the execution
of the runnable.<br/>
```
Task_10ms()
{
    Rte_IRead_Runnable_1_Data_A(&copy_A); Read a copy data of A before invoking Runnable_1
    Runnable_1() ; Call Runnable_1
}
```

A VariableAccess in the dataReadAccess role references the VariableDataPrototype.<br/>


## 1.2 Explicit data read access
The RTE generator creates a **blocking or non-blocking** API call to enable a receiver to poll (and read) data. This receive mode is an "explicit" mode since an explicit API call is invoked by the receiver.<br/>
-If the call is non-blocking (i.e. there is a VariableAccess in the dataReceivePointByValue or dataReceivePointByArgument role referencing the VariableDataPrototype for which the API is being generated, but no WaitPoint referencing a DataReceivedEvent
which references the VariableDataPrototype for which the API is being generated), the API call immediately returns the next value to be read and, if the communication is queued (event reception), it removes the data from the receiver-side queue.<br/>
-In contrast, a blocking call (i.e. the VariableDataPrototype, referenced by a VariableAccess in the role dataReceivePointByArgument, and for which the API is being generated, is referenced by a DataReceivedEvent which is itself referenced by a WaitPoint) will suspend execution of the caller until new data arrives (or a timeout occurs) at the according port. When new data is received, the RTE resumes the execution of the waiting runnable.


A VariableAccess in the dataReceivePointByValue or dataReceivePointByArgument role references the VariableDataPrototype.<br/>