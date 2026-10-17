# Dual-Axis Solar Tracker: A Model-Based Design Study in Simulink

![MATLAB](https://img.shields.io/badge/MATLAB-Simulink-orange)
![Stage](https://img.shields.io/badge/stage-model--in--the--loop-blue)
![Status](https://img.shields.io/badge/status-work%20in%20progress-yellow)

## Abstract

This repository contains a system-level Simulink model of a small dual-axis solar tracker with battery charging. The reference system comprises a 6 W photovoltaic (PV) panel, a single-cell (1S) Li-ion battery, and a 5 V load. The work follows a model-based design (MBD) approach and is currently at the **model-in-the-loop (MIL)** stage. Controller code generation is planned as a later step.

## Methodology

Two constraints govern the development process:

1. **Library-only implementation.** Models are built exclusively from Simulink library blocks. MATLAB Function blocks and hand-written code are not used.
2. **Verify-then-integrate.** Each subsystem is compared against independently computed reference values before it is connected to the rest of the model.

## Implemented Subsystems

| # | Subsystem | Description | Reference check |
|---|-----------|-------------|-----------------|
| 1 | Solar geometry | Hour angle, solar elevation, azimuth, and a direct normal irradiance (DNI) model with a cloud event | Elevation at solar noon: 54.73° (latitude 30.97° N, early October) |
| 2 | Incidence and irradiance | Incidence angle and plane-of-array (POA) irradiance | Hand calculation |
| 3 | PV model | Single-diode model with a vectorized 200-point I–V sweep | Maximum power of about 4.8 W at solar noon |
| 4 | Power chain | 5 V buck stage; CC/CV charger with pre-charge and recharge hysteresis; DW01A-style protection (OV/UV/OC); 3 Ah Li-ion state-of-charge (SOC) model; boost stage with load profile | Subsystem-level checks |
| 5 | Position control | Motor model (dead zone, first-order lag, travel limits); 12-bit encoder quantization; discrete PID with anti-windup | Step response |
| 6 | Current sensing | ACS712 and 10-bit ADC model | Resolution of about 26 mA per count |

### Notes on the reference values

- **Solar elevation.** At solar noon the elevation is
  `α = 90° − |φ − δ|`,
  where φ is the latitude and δ the solar declination. For φ = 30.97° N, α = 54.73° corresponds to δ ≈ −4.3°, which is consistent with early October.
- **Current-sensing resolution.** For a 10-bit ADC with a 5 V reference, one count is 5 V / 1024 ≈ 4.88 mV. With the 5 A variant of the ACS712 (185 mV/A), this gives about 26 mA per count.

## Roadmap

- [x] Solar geometry and irradiance
- [x] PV model
- [x] Power chain (buck, charger, protection, battery, boost)
- [x] Position control and sensing chain
- [ ] Sensor noise models. The ACS712 noise level relative to the 26 mA quantization step affects the detectability of power changes in the tracker.
- [ ] Perturb-and-observe (P&O) controller implemented as a Stateflow chart. Here P&O perturbs the **panel angle** and observes the resulting power. It is not an electrical MPPT of the converter duty cycle.
- [ ] Time-compressed 12-hour comparison of three cases: fixed panel, P&O tracker, and ideal tracker.
- [ ] Controller code generation (later stage).

## Repository Structure

```text
.
├── models/        % Simulink models (.slx)
├── scripts/       % Reference calculations and verification scripts
├── data/          % Reference values and simulation outputs
├── docs/          % Figures and notes
└── README.md
```

> Adjust the folder names above to match the actual repository layout.

## Requirements

- MATLAB and Simulink (specify the release used)
- Stateflow (for the planned P&O controller)

## Usage

1. Clone the repository.
2. Open MATLAB and set the repository root as the current folder.
3. Open the model in `models/` and run the verification scripts in `scripts/`.

## Limitations

- The study is simulation-only. No hardware validation has been performed.
- Maximum PV power depends on the assumed irradiance and cell temperature, and is not quoted at standard test conditions (STC).

## Feedback

Comments are welcome, particularly from practitioners with experience in P&O tracking algorithms and small-scale battery management systems. Please open an issue or a discussion.

## Citation

If you use this work, please cite the repository:

```bibtex
@misc{dual_axis_tracker_simulink,
  title  = {Dual-Axis Solar Tracker: A Model-Based Design Study in Simulink},
  author = {<Author name>},
  year   = {2026},
  url    = {<repository URL>}
}
```

## License

Specify a license, for example MIT.
