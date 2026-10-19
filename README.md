# ACC Controller on Infineon AURIX TC387

> Modern Control course · Infineon × Kookmin Univ. ACC Development Competition · 2025.09 – 2025.12 · 3-person team
> **4th place out of 14 teams**

An Adaptive Cruise Control (ACC) logic designed in MATLAB/Simulink, auto-generated into C code and deployed on an Infineon AURIX TC387. The controller is verified step by step from MIL (Model-in-the-Loop) to PIL (Processor-in-the-Loop) against a PreScan virtual vehicle.

---

## System Architecture

```mermaid
flowchart LR
    A["PC · PreScan<br/>virtual road, vehicle,<br/>range sensor"] -- "sensor data (TCP/IP)" --> B["AURIX TC387<br/>ACC control code<br/>(Embedded Coder C)"]
    B -- "throttle / brake (TCP/IP)" --> C["PC · PreScan<br/>virtual vehicle driving"]
```

| Item | Detail |
|---|---|
| Modeling | MATLAB / Simulink |
| Code generation | Embedded Coder (Simulink → C) |
| Target MCU | Infineon AURIX TC387 (TriCore) |
| Simulation | PreScan (vehicle, road, sensors) |
| Interface | Ethernet TCP/IP between PreScan and TC387 |
| Timing | STM (System Timer) based execution time measurement |

<p align="center">
  <img src="docs/simulink_acc_model.png" width="760" alt="Simulink ACC model block diagram">
  <br><sub>Simulink ACC model: sensor input → MPC / NSCC control blocks → throttle / brake output</sub>
</p>

---

## Control Algorithms

### MPC (Model Predictive Control)
- Predicts relative distance and speed to the preceding vehicle over a horizon and minimizes a cost function.
- Maximum speed constraint included; solved as a QP with the KWIK solver.

### PID
- Initial controller and performance baseline for MPC.

### NSCC (Navigation-based Smart Cruise Control, rule-based)
- In corners, the range sensor can lose the preceding vehicle.
- The target speed is adjusted from the map curvature so the gap is kept while the sensor is lost.

---

## Verification Flow (MIL → PIL)

| Stage | Method |
|---|---|
| MIL | Simulink model with PreScan; signals observed with Scope per section |
| Code generation | Embedded Coder C code built for TC387 |
| PIL | Generated code on TC387 in the loop with PreScan; RX/TX execution time measured with STM |

**Design note**: The TC387 FPU supports single precision only. `double` arithmetic falls back to software emulation, so the computation and communication interface types are implemented in `float`.

---

## Results

<p align="center">
  <img src="docs/prescan_scenario.jpg" width="380" alt="PreScan driving scenario">
  <img src="docs/distance_tracking_mpc.png" width="380" alt="Relative distance tracking with PID and MPC">
  <br><sub>Left: PreScan scenario with curved sections · Right: relative distance tracking (PID + MPC)</sub>
</p>

- Steady-state error of ACC **below 13%**
- Stable re-convergence after sensor loss in corners with NSCC, compared with PID only
- 4th place out of 14 teams

---

> Competition source code will be added after cleanup.
