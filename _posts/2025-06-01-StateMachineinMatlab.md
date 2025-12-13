## State Machine Outline
Usually, the software state will be the same as the system state when we derive the system state into a software state (but not always). Remember that our software runs in discrete time, which means a function or piece of code in the application will eventually be invoked by the OS at a defined sampling time. The same applies to a state machine; it can be defined inside a function to perform logic for running a specific state at a point in time and handling state transitions. Each state has 3 parts:<br>
+ **Entry**: the action will be run only once when the state is entered from another state<br>
+ **During (or Action)**: the action will run continuously until the state machine matches the condition for a state transition to move to another state.<br>
+ **Exit**: the action will be run before moving to another state.<br>

A state machine manages many states, and one state can move to another when a specific condition is met; this is called a **state transition**.

## State machine in MATLAB Simulink
So now we understand what a state machine is and its layout.

![State machine example](https://github.com/lexma1412/lexma1412.github.io/blob/main/assets/StateMachine.png?raw=true)<br/>

Back to the example with 3 states: Standby, Ready, and Running. The condition for the state transition from Standby to Ready is that there is no error in the system. The condition for the state transition from Ready to Running is after 2 seconds. The condition for the state transition from Running to Standby is when an error occurs. The above image shows the implementation in MATLAB.