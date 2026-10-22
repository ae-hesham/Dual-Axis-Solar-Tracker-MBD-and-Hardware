<div align="center">

# ☀️ Dual-Axis Solar Tracker

### A Model-Based Design Study in Simulink

*A system-level model of a small dual-axis solar tracker with battery charging, built with a library-only, code-generation-ready approach.*

<br>

[![MATLAB](https://img.shields.io/badge/MATLAB-Simulink-orange?style=for-the-badge&logo=mathworks&logoColor=white)](https://www.mathworks.com/)
[![Stage](https://img.shields.io/badge/Stage-Model--in--the--Loop%20(MIL)-blue?style=for-the-badge)](#-project-roadmap)
[![Status](https://img.shields.io/badge/Status-Work%20in%20Progress-yellow?style=for-the-badge)](#-project-roadmap)
[![License](https://img.shields.io/badge/License-Proprietary-red?style=for-the-badge)](LICENSE)

<br>

<img src="models/1.gif" alt="Dual-Axis Solar Tracker 3D Simulation" width="90%" />

<sub><b>Figure 1</b> · 3D visualization of the tracker. The panel continuously adjusts azimuth and elevation relative to the sun while streaming live charge/discharge signal flows and diagnostic telemetry.</sub>

</div>

---

## 📑 Table of Contents

- [Abstract](#-abstract)
- [System Architecture](#-system-architecture)
- [Design Methodology](#%EF%B8%8F-design-methodology)
- [Implemented Subsystems](#-implemented-subsystems)
- [Mathematical Reference](#-mathematical-reference)
- [Project Roadmap](#%EF%B8%8F-project-roadmap)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [Limitations](#%EF%B8%8F-limitations)
- [Feedback](#-feedback)
- [Citation](#-citation)
- [License](#-license)

---

## 📌 Abstract

This repository contains a system-level Simulink model of a small dual-axis solar tracker with battery charging. The reference system comprises a **6 W photovoltaic (PV) panel**, a **single-cell (1S) Li-ion battery**, and a **5 V load**.

The work follows a **model-based design (MBD)** approach and is currently at the **model-in-the-loop (MIL)** stage. Controller code generation is planned as a later step.

### Reference System at a Glance

| Component | Specification |
|:--|:--|
| ☀️ PV panel | 6 W |
| 🔋 Battery | 1S Li-ion, 3 Ah |
| 🔌 Load | 5 V |
| 🧪 Current stage | Model-in-the-Loop (MIL) |

---

## 📐 System Architecture

<p align="center">
  <img src="docs/system_model.png" alt="Top-level Simulink model" width="100%" />
</p>

<p align="center">
  <sub><b>Figure 2</b> · Top-level Simulink block diagram.</sub>
</p>

Signal flow propagates from environmental solar dynamics (left) through the PV array and 5 V buck stage to the **CN3791** charger, **DW01A / FS8205A** protection ICs, the 3 Ah Li-ion cell model, and the boost output stage (right). The lower loops handle closed-loop position control (motors, encoders, PID) and **INA219** sensor telemetry over I2C.

---

## 🛠️ Design Methodology

Development adheres strictly to two engineering constraints:

| | Principle | Description |
|:-:|:--|:--|
| **1** | **Strict library-only implementation** | Built exclusively with standard Simulink library blocks. No MATLAB Function blocks or hand-written scripts are used within the simulation loop, guaranteeing full compatibility with code generators. |
| **2** | **Verify-then-integrate workflow** | Every subsystem is independently verified against baseline mathematical calculations before system integration. |

---

## ⚡ Implemented Subsystems

| # | Subsystem | Functional Description | Reference Verification |
|:-:|:--|:--|:--|
| **1** | **Solar Geometry** | Calculates hour angle, solar elevation, azimuth, and Direct Normal Irradiance (DNI), including cloud transient events. | Elevation at solar noon: **54.73°** (latitude 30.97° N, early October) |
| **2** | **Incidence & Irradiance** | Computes incidence angle and Plane-of-Array (POA) irradiance. | Verified against analytical geometry calculations |
| **3** | **PV Array Model** | Single-diode model with a vectorized 200-point I–V curve sweep. | Peak power ≈ **4.8 W** at solar noon |
| **4** | **Power Chain** | 5 V buck stage, CC/CV CN3791 charger model (pre-charge and recharge hysteresis), DW01A-style OV/UV/OC protection, 3 Ah Li-ion SOC estimator, and boost output load converter. | Subsystem-level energy balance checks |
| **5** | **Position Control** | Actuator models (dead zone, 1st-order lag, mechanical stops), 12-bit encoder quantization, and discrete PID with anti-windup. | Transient step-response analysis |
| **6** | **Sensing & Telemetry** | INA219 current/voltage monitor over I2C (shunt and bus registers, 12-bit ADC, polling latency). | LSB resolution: **10 µV / R<sub>shunt</sub>** (100 µA step for 0.1 Ω) and **4 mV** bus step |

---

## 🧮 Mathematical Reference

### Solar Elevation Angle (α)

Calculated at solar noon as:

$$
\alpha = 90^\circ - \lvert \phi - \delta \rvert
$$

where $\phi = 30.97^\circ\,\text{N}$ is the latitude and $\delta \approx -4.3^\circ$ is the solar declination, matching early October.

### Current Sensor Resolution

$$
\Delta I = \frac{10\,\mu\text{V}}{R_{shunt}}
$$

For an assumed $R_{shunt} = 0.1\,\Omega$, this yields a discrete resolution of **100 µA/LSB**. Dual INA219 modules (PV and battery lines) use distinct I2C bus addresses.

---

## 🗺️ Project Roadmap

**Completed**

- [x] Solar geometry & irradiance modeling
- [x] Photovoltaic (PV) single-diode model
- [x] Complete power chain (buck, charger, protection, battery SOC, boost)
- [x] Position control loops & INA219 sensor telemetry

**Planned**

- [ ] **Sensor noise & non-idealities:** INA219 offset, conversion/averaging latency, and I2C jitter effects on tracker convergence.
- [ ] **Stateflow P&O tracker:** Perturb-and-Observe (P&O) algorithm controlling **panel orientation angles** (mechanical MPPT).
- [ ] **Comparative performance analysis:** 12-hour time-compressed comparison of
  1. Fixed panel setup
  2. Mechanical P&O tracker
  3. Ideal mathematical tracker
- [ ] **Embedded target code generation (PIL/HIL).**

---

## 📂 Repository Structure

```text
.
├── models/     # Simulink model files (.slx) & animations (.gif)
├── scripts/    # Analytical verification and baseline calculations
├── data/       # Simulation logs, datasets, and reference parameters
├── docs/       # High-resolution block diagrams and technical documentation
├── LICENSE     # Legal terms, proprietary rights, and license conditions
└── README.md   # Project documentation
```

---

## 🚀 Getting Started

### Requirements

- MATLAB and Simulink *(specify the release used)*
- Stateflow *(for the planned P&O controller)*

### Usage

1. Clone the repository.
2. Open MATLAB and set the repository root as the current folder.
3. Open the model in `models/` and run the verification scripts in `scripts/`.

---

## ⚠️ Limitations

> [!NOTE]
> This is a simulation-only study at the MIL stage.

- No hardware validation has been performed.
- The I2C link is modelled as a fixed read latency with register quantization. Bus faults and NACKs are not modelled.
- Maximum PV power depends on the assumed irradiance and cell temperature, and is not quoted at standard test conditions (STC).

---

## 💬 Feedback

Comments are welcome, particularly from practitioners with experience in **P&O tracking algorithms** and **small-scale battery management systems**. Please open an issue or a discussion.

---

## 📖 Citation

If you use this work, please cite the repository:

```bibtex
@misc{dual_axis_tracker_simulink,
  title  = {Dual-Axis Solar Tracker: A Model-Based Design Study in Simulink},
  author = {Abdelrahman Hesham},
  year   = {2026},
  url    = {https://github.com/ae-hesham/Dual-Axis-Solar-Tracker-MBD-and-Hardware}
}
```

---

## 📜 License

This project is licensed under the Proprietary License. See the [LICENSE](LICENSE) file for details.

<div align="center">
<br>
<sub>Built with Simulink · Model-Based Design · Made by Abdelrahman Hesham</sub>
</div>
