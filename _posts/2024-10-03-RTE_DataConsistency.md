# Data Consistency 
Concurrent accesses to shared data memory can cause data inconsistencies.
In general this must be taken into account when several code entities accessing the same data memory are running in different contexts - in other words when systems using parallel (multicore) or concurrent (singlecore) execution of code are designed. 
More general: Whenever task context-switches occur and data is shared between tasks or ISR2s, data consistency is an issue.</br> 
AUTOSAR systems use operating systems according to the AUTOSAR-OS specification which is derived from the OSEK-OS specification. The Autosar OS specification defines a priority based scheduling to allow event driven systems. This means that tasks with higher priority levels or ISR2s are able to interrupt (preempt) tasks with lower priority level. 

![Data inconsistency example from RTE Autosar](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/RTE_DataConsistency/example.png?raw=true)

The inconsistencies of data can happen in following interaction/communication cases: 
|Number|Case|Description|Prevent Mechanism| 
|:----|:----|:-----------|:----------------| 
|1|Communication within one atomic AUTOSAR SW-C|Communication between Runnables of one atomic AUTOSAR SW-C running in different task/ISR2 contexts </br> where communication between these Runnables takes place via commonly accessed data. If the need to support data consistency by the RTE exists, </br> it must be specified by using the concepts of "ExclusiveAreas" or "InterRunnableVariables" only.|-ExclusiveAreas(Interrupt blocking strategy,  Usage of OS resources) <br/> -InterRunnableVariables(Copy strategy with implicit behaviour) <br/> -Sequential scheduling strategy <br/> -Task blocking strategy| 
|2|Intra-partition communication between AUTOSAR SW-Cs|Sender/Receiver (S/R) communication between Runnables of different AUTOSAR SW-Cs using implicit or explicit data exchange can be realized by the RTE through commonly accessed RAM memory areas. <br/> Data consistency in Client/Server (C/S) communication can be put down to the same concepts as S/R communication. Data access collisions must be avoided. The RTE is responsible for guaranteeing data consistency|-Copy strategy (RTE implicit behaviour write/read port)| 
|3|Inter-Partition communication|The RTE has to guarantee data consistency|Inter partition data communication using IOC</br> Inter partition data communication using Basic Software Scheduler </br> Accessing (Ld)Com and Det in multicore/multipartition configuration</br> Accessing NvM in multicore/multipartition configurations </br>Signaling and control flow support for inter partition communication </br>Trusted Functions </br> Memory Protection and Pointer Type Parameters in RTE API| 
|4|Intra-ECU communication between AUTOSAR SW-Cs and BSW modules with AUTOSAR interfaces|This is a special case of the above two.|Refer above two| 
|5|Inter ECU communication|COM has to guarantee data consistency for communication between ECUs on complete path between the COM modules of different ECUs. The RTE on each ECU has to guarantee that no data inconsistency might occur when it invokes COM send respectively receive calls supplying respectively receiving data items which are concurrently accessed by application via RTE API call, especially when queueing is used since the queues are provided by the RTE and not by COM.|No specify| 

Let go through each mechanism in following sections. 

## Exclusive Areas 
As mention above, Exclusive Areas is used to prevent Data Inconsistency between Runnables of one atomic AUTOSAR SW-C running in different task/ISR2 contexts, also for the Basic Software Modules want to protect its inside data against multi API access. The concept of ExclusiveArea is more a working model, it is not a particular mechanism.
**Focus of the ExclusiveArea concept is to block potential concurrent accesses to get data consistency. ExclusiveAreas implement critical section.** 
* If an AUTOSAR SW-C requests the RTE to look for data consistency for it’s internally used data:  use the API calls "Rte_Enter()" and "Rte_Exit() defines the begin and the end of the code sequence containing data accesses the RTE. Similar for Basic Software Module, but it uses "SchM_Enter() and "SchM_Exit()". 
E.g: Runnable_1 want to access resource Y exclusively 

``` 
Runnable_1():   
Rte_Enter()   
Access (read/modify/write) resource Y   
Rte_Exit() 
``` 
* If the SW designer wants to have the mutual exclusion for complete RunnableEntitys: can specify this by using the ExclusiveArea in the role "runsInside" in the AUTOSAR SW-C description. 
Basic Software Module does not support case runinside ExclusiveArea. The assignment data consistency mechanism of ExclusiveArea is configure via **RteExclusiveAreaImplMechanism** parameter with value: ALL_INTERRUPT_BLOCKING/OS_INTERRUPT_BLOCKING/OS_RESOURCE/OS_SPINLOCK/NONE/RTE_PLUGIN corresponding with Interrupt blocking strategy/Usage of OS resources mechanism. 

## InterRunnableVariables
InterRunnableVariables mechanism is one kind of copy strategy and be used to guarantee data consistency for communication between Runnables of one AUTOSAR. Simply, when exchange global data between Runnables,we need make sure the data inconsistency problem not happen.
InterRunnableVariables has 2 types:
1. **InterRunnableVariables with implicit behavior (implicitInterRunnableVariable)**: Focus of InterRunnableVariable with implicit behavior is to avoid concurrent accesses by redirecting second, third, accesses to data item copies.<br/>
-The Runnable IN data is stable during Runnable execution, which means that during an Runnable execution several read accesses to an implicitInterRunnableVariable always deliver the same data item value.<br/>
-The Runnable OUT data is forwarded to other Runnables not before
Runnable execution has terminated, which means that during an Runnable execution write accesses to implicitInterRunnableVariable are not visible to other Runnables.

2. **InterRunnableVariables with explicit behavior (explicitInterRunnableVariable)**: Focus of InterRunnableVariables with explicit behavior is to block potential concurrent accesses to get data consistency.<br/>
-This mechanism is similar to Exclusive Areas, however autosar define with scope of inside one SWC only for convenient use.


|InterRunnableVariables with implicit behavior|InterRunnableVariables with explicit behavior|
|:----|:----|
|- In applications with very high SW-C communication needs and much real time constraints (like in powertrain domain) the usage of a copy mechanism to get data consistency might be a good choice because during RunnableEntity execution no data consistency overhead in form of concurrent access blocking code and runtime during its execution exists.- independent of the number of data item accesses. <br/> - Disadvantage is RAM can be high due to variable copy.          |-Application require to have access to the newest data item value without any delay, even several times during execution of a Runnable. <br/>-Focus of InterRunnableVariables with explicit behavior is to block potential concurrent accesses to get data consistency.<br/> - Disadvantage is timing constraint.|


## Other mechanisms
* Sequential scheduling strategy:<br/>
-The activation code of Runnables is sequentially placed in one task/ISR2 so that
no interference between them is possible because one Runnable is only activated
after the termination of the other. Data consistency is guaranteed. In other words, it is just the way to put the runnable in one task with order, depend on design.
```
Task_001():
    Runnable_1():
        Access read/write data_1
    Runnable_2():
        Access read/write data_1
```
* Interrupt blocking strategy:<br/>
-Interrupt blocking can be an appropriate means if collision avoidance is required
for a very short amount of time. This might be done by disabling respectively
suspending all interrupts, Os interrupts only or - if hardware supports it - only
of some interrupt levels. In general this mechanism must be applied with care
because it might influence SW in tasks with higher priority too and the timing of
the complete system.
* Usage of OS resources:<br/>
-Usage of OS resources. Advantage in comparison to Interrupt blocking strategy is that less SW parts with higher priority are blocked. Disadvantage is that
implementation might consume more resources (code, runtime) due to the more
sophisticated mechanism. Appropriateness of this mechanism may vary depending on the number of OSs/cores and/or the number of available resources.
* Task blocking strategy:<br/>
-Mutual task preemption is prohibited. This might be reached e.g. by assigning
same priorities to affected tasks, by assigning same internal OS resource to affected tasks or by configuring the tasks to be non-preemptive. This mechanism may be inappropriate in multi-partitioned systems.

# Mechanism for partition data consistency
TBD