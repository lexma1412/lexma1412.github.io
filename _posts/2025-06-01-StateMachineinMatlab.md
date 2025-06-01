---
layout: default
title:  "State Machine"
---

# State Machine and Design with MATLAB Simulink

In complex applications, we usually divide the operation of a system into many states or modes. For example, in a motor control system, we define a Standby state where the system does not run, a Ready state where the system performs related tests to confirm everything is OK, and a Running state where the motor operates. A system consists of software and hardware; both should maintain the system's states, and the responsibility for state management is defined in the software/hardware requirements.

This post introduces the state machine concept in software.

## State Machine Outline
Usually software state will be the same system state hen we derive system state into software state (but not always). Remember that our software runs in discrete time, which means a function or piece of code in the application will eventually be invoked by the OS at a defined sampling time. The same applies to a state machine; it can be defined inside a function to perform logic for running a specific state at a point in time and handling state transitions. Each state has 3 parts:<br>
+ **Entry**: the action will be run only once when the state is entered from another state<br>
+ **During (or Action)**: the action will run continuously until the state machine matches the condition for a state transition to move to another state.<br>
+ **Exit**: the action will be run before moving to another state.<br>

State machine manages many states, and one state can move to another when a specific condition is met; this is called a **state transition**.