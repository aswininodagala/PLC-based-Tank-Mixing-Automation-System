*This project is done with an understanding of how control logics work in industries using ladder logic programming.*
 
 Project Report
PLC Tank Mixing Control System

	Date: May 11, 2026
 


    1. Project Overview and Objective
The objective of this project is to design, implement, and test an automated ladder logic program to control a liquid mixing system within a batch processing tank. The system sequentially feeds two distinct materials (Material A and Material B) into a central tank, mixes them for a specified duration using an electrically driven agitator, and then drains the final blended product through an outlet valve. The automation is governed by a Programmable Logic Controller (PLC) utilizing discrete digital inputs from level sensors and discrete digital outputs to actuate valves and the motor.


    2. System Architecture & Components
The automated mixing tank relies on several key physical elements, interfaced directly with the PLC modules:
Level Sensors: Three distinct capacitive or optical level switches are utilized to detect fluid boundaries. A Low Level Switch (LLS) identifies an empty tank, Level Switch A detects when the first material has reached its target volume, and Level Switch B detects when the tank is full.
Control Valves: Single-acting piston valves are used for fluid routing. They operate in a strictly binary fashion (fully open or fully closed). Inlet Valve 1 handles Material A, Inlet Valve 2 handles Material B, and the Outlet Valve manages the drainage of the final product.
Agitator Motor: A mechanically coupled agitator ensures uniform blending of the materials. It is timed via the PLC's internal timer instructions.




    3. Input / Output (I/O) Allocation
The following table defines the addressing scheme used within the LogixPro simulation environment for all peripheral devices and internal logic tags.

Address

Signal Type

Description / Physical Element

I:1/14

Input (Pushbutton)

Master Start

Address

Signal Type

Description / Physical Element

I:1/15

Input (Pushbutton)

Master Stop (Normally Closed configuration)

I:1/2

Input (Sensor)

Low Level Switch (LLS) - Detects empty tank

I:1/1

Input (Sensor)

Level of Material A

I:1/0

Input (Sensor)

Level of Material B (Tank Full)

O:2/4 (or B3:0/0)

Internal Relay Bit

Master Coil / System Enable

O:2/0

Output (Actuator)

Inlet Valve 1 (Material A Feed)

O:2/1

Output (Actuator)

Inlet Valve 2 (Material B Feed)

O:2/2

Output (Motor)

Agitator Motor (Mixing)

O:2/3

Output (Actuator)

Outlet Valve (Product Drain)

    4. Ladder Logic Program Description

Rung 000: Master Control Circuit
This rung establishes the core safety and operational state of the machine. It utilizes a standard latching circuit. When the Start pushbutton (I:1/14) is engaged, the Master Coil (O:2/4) energizes. A parallel seal-in contact of O:2/4 ensures the system remains active once the start button is released. The Stop pushbutton (I:1/15) is placed in series to safely break the circuit and halt operations immediately upon pressing.
Rung 001: Material A Fill Cycle
Governs Inlet Valve 1 (O:2/0). The valve opens under the condition that the Master Coil is active, and the tank indicates it requires filling. It is triggered by the Low Level Switch (I:1/2) or the initial Start press, latching in via its own O:2/0 contact. The fill process continues until the liquid reaches the Material A threshold, at which point the Normally Closed (NC) contact of Level Material A (I:1/1) opens, cutting power to the valve.
Rung 002: Material B Fill Cycle
Governs Inlet Valve 2 (O:2/1). Once Material A has reached its setpoint, the Normally Open (NO) contact for Level Material A (I:1/1) closes, initiating the flow of Material B. This rung also utilizes a seal-in circuit. Flow halts when the liquid reaches the top sensor, Level Material B (I:1/0), causing its NC contact to open and de-energize O:2/1.
Rung 003 & 004: Agitation and Mixing Delay
These rungs control the kinetic mixing phase. Once the tank is completely full (Level Material B, I:1/0, is true), the Agitator Motor (O:2/2) turns on. Concurrently, rung 004 powers an On-Delay Timer (TON) at address T4:0. The timer tracks the mixing duration (configured for a 50 accumulated value at a 0.1s time base, equaling 5 seconds of mixing). Once the accumulated value matches the preset, the T4:0/DN (Done) bit toggles, stopping the motor via its NC contact in rung 003.
Rung 005 & 006: Drainage and Batch Tracking
Rung 005 commands the Outlet Valve (O:2/3). It requires the mixing cycle to be entirely finished (T4:0/DN is true). The valve latches open to drain the tank. It remains open until the fluid level drops below the Low Level Switch (I:1/2), breaking the latch. Rung 006 features an Up-Counter (CTU) at C5:0, incrementing its accumulated value every time the Outlet Valve cycles, effectively keeping a log of successfully completed batches.


6. Conclusion
The developed ladder logic effectively automates the entire batch mixing operation without the need for human intervention between cycles. By utilizing internal timers and a sequential latching structure driven by the physical fluid levels, the PLC strictly adheres to the process requirements. Furthermore, the inclusion of an automated batch counter ensures production tracking is inherently tied into the control system.
