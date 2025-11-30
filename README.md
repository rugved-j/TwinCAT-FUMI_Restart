# Master's Thesis for the topic 'Development of a multi-purpose robotics system for flexible task automation' 

A public, work-sample repository showcasing a **production-style pick–place and solid dosing cell** built around a **UR7e** robot, a **Beckhoff** PLC/IPC running **TwinCAT 3**, and a **closed-loop dosing process** using a load cell. It highlights **state-based control**, **tool changing with a SCHUNK station**, and **industrial comms (PROFINET)** for streaming 6D poses to the robot.

> This repo is intended as a portfolio artifact: it demonstrates engineering depth (controls, comms, robotics integration) and clarity (documentation, structure, testability).
> 

---

## What this project demonstrates

* **Deterministic, state-based orchestration** (UML Statecharts / PLC ST) for pick, place, dosing, and recovery.
* **UR7e integration over PROFINET**: cyclic O2T pose streaming (`p[x,y,z,rx,ry,rz]`) and T2O handshakes (Busy/Done/Error/Seq).
* **End-effector flexibility** via a **SCHUNK gripper changing station** (automatic tool ID, on-the-fly TCP/payload updates).
* **Closed-loop dosing**: stepper-driven auger/feeder with **PID control from load-cell feedback** (WTB Analog → Beckhoff AI).
* **Industrial motion**: Beckhoff EL7062 stepper terminal (enable, scaling, safety), homing and indexed moves.
* **Operator visibility**: ring buffers for recent IDs/poses, status LEDs/vars, and an Image/Process watch stream.

---

## System architecture (high level)

* **Controller**: Beckhoff IPC (TwinCAT 3.1.4026.x)

  * PLC (Structured Text) + **TF1910 UML Statecharts**
  * Optional: **TF5xxx NC-PTP** (for motion primitives) and **Drive Manager 2** for stepper
  * **TF6220 PROFINET Controller** for UR device comms
* **Robot**: **UR7e** (URScript & PROFINET device)
* **Drives/IO**:

  * **EL7062** stepper terminal (dosing / index positioning)
  * **EL3078** analog input (0–10 V or 4–20 mA) from **Bosche WTB Analog** load-cell amplifier
  * Digital IO for interlocks and station signals
* **End-Effectors**: SCHUNK quick-change station + finger sets/spatula tool

---

## Key workflows

### Pick & Place (reference task)

* Home → Approach pick → Align → Grip (force/position limits) → Transfer → Approach place → Release → Retreat
* **Statechart** governs sequence and interlocks; handshakes ensure each action completes before transition
* Fault/timeout transitions funnel to **Recover**, then return via **History** to the interrupted state

### Solid Dosing (closed loop)

* Bulk feed (open-loop) → Fine feed (**PID on live weight**) → Hold & verify → Done
* **EL7062** indexes to loading/dispense angles; **WTB Analog** provides mass feedback to PLC PID
* Tolerance windows and dwell/stability checks prevent overshoot and ensure repeatability

### Tool change (SCHUNK)

* Dock → Lock → Tool ID verify → Update TCP/payload/inertia → Safety limits → Undock
* Enables rapid switching between **gripper fingers AB** and **spatula (tool C)** without manual intervention

---

## Repository contents

> Note: Some files are example/stripped work artifacts to share techniques without exposing proprietary assets.

* `PLC/`

  * `MAIN.TcPOU` – main program/statechart call & orchestration
  * `FB_Camera/` – perception/pose packaging helpers (if included)
  * `FB_Motor/` – stepper + PID dosing logic
  * `FB_Robot/` – sequence handshake, safety/interlocks
  * `Types/` – DUTs for pose, command enums, config structs
* `Docs/`

  * `figures/` – pipeline & state diagrams (placeholders)
  * `readme_diagrams.md` – quick visual overview
* `Configs/`

  * Example GSDML/DUT mapping notes for UR PROFINET device
  * I/O mapping snapshots (EL7062, EL3078)
---

## Getting started (engineering PC)

1. **Clone** this repository.
2. Open the TwinCAT solution in **TcXaeShell/VS** (TwinCAT 3.1.4026.x).
3. Ensure required options are present (trial is fine):

   * **TF1910 UML**, **TF6220 PROFINET**, (optional) **TF5xxx NC-PTP**
4. **Set AMS route** to your IPC (or run locally), then:

   * Activate configuration (Config → Run)
   * Map PROFINET device to the **UR controller** and link O2T/T2O structs
   * Map EL7062 stepper and EL3078 analog channels; set ranges (4–20 mA or 0–10 V)
5. **Build → Login → Run**. Start the UR program that consumes the pose/command registers.

> **Safety**: Keep speeds low on first runs, verify TCP/payload on tool change, and confirm safe zones/interlocks.

---

## Highlights for reviewers (what to look at)

* **Statechart design**: clean separation of **Entry/Do/Exit** actions, guarded transitions, and **History-based** recovery.
* **Handshake discipline**: every robot/drive action has **Busy/Done/Error** semantics; no blind sequencing.
* **Closed-loop process**: dosing shows practical control (bulk→fine, PID tuning, stability gating).
* **Extensibility**: SCHUNK tool change path proves the cell scales to new parts/tools with minimal code churn.

---

## Contact

If you’re evaluating this for a role:

* I’m comfortable owning **spec → implementation → commissioning**, across PLC/robot comms, motion, perception, and safety interlocks.
* Happy to discuss design decisions, trade-offs, and how this architecture generalizes to your environment.

