# Physics-Informed Robot Condition Monitoring
### KUKA KR16 L6 · Dynamics, time series and Healthy-state references

**Mounir Belboualia** · Independent personal research project  
[Research enquiries](mailto:belboualia.mounir@tha.de)

> **Can combining physical robot dynamics with statistical time-series models produce a reliable Healthy-state reference and improve condition monitoring compared with purely data-driven methods?**

This project investigates that question from the equations of motion to the behaviour of real sensor signals. The aim is an interpretable reference for normal robot operation: a model whose prediction errors can be examined before they are used to flag changes in condition.

**Current conclusion:** adding physics has not yet demonstrated a practically convincing advantage over a simple autoregressive reference on native acceleration signals. Short-term prediction accuracy also did not translate into reliable anomaly thresholds. These negative findings help identify what the next experiment must resolve.

## Research approach

| Perspective | Question investigated |
|---|---|
| **Newtonian mechanics and numerical modeling** | Can integrated joint dynamics and rigid-body sensor-point acceleration explain the measurements? |
| **Motor-current proxies** | Do measured current signals add predictive information, and how do delays and smoothing affect their interpretation? Current is treated as a proxy, not measured joint torque. |
| **Sensor orientation and acceleration** | What can covariance, spectra and PCA reveal about signal geometry? Sensor mounting and the measurement chain are examined separately from an empirical signal plane. |
| **Video-based kinematics** | Can image observations provide an independent motion reference? Camera calibration and synchronization limit quantitative conclusions. |
| **Coupled Euler–Lagrange dynamics** | Are the coupled joint equations, constraints and moment transformations consistent with an independent Newton–Euler formulation? |
| **Autoregressive and hybrid Gray-Box models** | Does a physical prediction plus statistical residual dynamics improve on direct time-series prediction? |
| **Anomaly detection** | Do thresholds calibrated on Healthy recordings remain reliable on later Healthy recordings? |

The studies compare physical, statistical and hybrid candidates with zero prediction and Healthy time templates. Chronological separation, declared scoring windows, frozen model settings and explicit assumptions support interpretation. Previously explored recordings remain development evidence—not fresh blind tests. Software checks and mathematical consistency are distinguished from experimental validation.

## What the experiments found

- **Filtered dynamics:** the corrected V2 study reduced RMSE by **3.13% relative to zero prediction** across ten QC-eligible test days. The advantage over a Healthy time template remained inconclusive. This result concerns filtered, offline targets.
- **Native acceleration:** in V5, the hybrid model C achieved **1.452393 m/s²** mean day RMSE versus **1.452605 m/s²** for direct AR model A—an improvement of approximately **0.0146%**. This is a numerical difference, not evidence of a practically meaningful physical contribution.
- **Prediction horizon matters:** that one-step result uses a horizon of approximately **39 microseconds**. At **10–500 ms**, the tested waveform predictors showed no practically relevant advantage over zero prediction.
- **Alarm calibration failed to transfer:** thresholds fitted on three calibration days were exceeded on **all twelve later recordings labelled Healthy**, for models A–D. A large anomaly score therefore cannot be interpreted as confirmed wear.
- **Geometry remains conditional:** low-frequency PCA identified a reproducible plane candidate for one J2 condition. It does not establish the mechanical sensor orientation or resolve the competing kinematic hypotheses.

V5 concerns J3 motion at 50% speed and the J3 accelerometer, with six training days, three calibration days and twelve previously known later Healthy days. Results from different bands, targets and evaluation protocols must not be compared as if they were one controlled experiment.

## Open questions and collaboration

The next step is to determine whether the main limitation lies in the physical observation model, synchronization, operating-condition drift or the statistical reference.

Particularly valuable directions are:

- **A synchronized reference experiment:** independent joint angles, documented sensor mounting and a characterized current/acceleration measurement chain.
- **Broader Healthy calibration:** independent days and operating conditions, with reserved test recordings and uncertainty assessed at recording or day level.
- **Alternative targets:** compare waveform prediction with band-energy or other physically interpretable summaries.
- **Condition-monitoring validation:** evaluate frozen methods against independently documented condition changes before making wear or maintenance claims.

I welcome research collaborations, internships and discussions in **Data Science, Machine Learning, Scientific Computing and physics-informed time-series modeling**. Contact: **[belboualia.mounir@tha.de](mailto:belboualia.mounir@tha.de)**.

## Explore the evidence

| Material | Contents |
|---|---|
| [Research report — LaTeX](main.tex) | Research progression, mathematical reasoning and limitations; compile with [preamble.tex](preamble.tex), [spectrum.png](spectrum.png) and [alarm.png](alarm.png) |
| [Healthy identification report](HEALTHY_REPORT.md) | Early models, corrections and identification limits |
| [V5 report](V5_REPORT.md) | Model comparisons, evaluation conditions and negative findings |
| [V5 summary](V5_SUMMARY.csv) · [Anomaly scores](V5_ANOMALY_SCORES.csv) | Selected numerical results |

This repository is a selected historical documentation archive, not a complete reproduction package. Some report references point to project files outside this archive. No calibrated sensor orientation, reliable wear diagnosis, remaining-useful-life estimate or deployment readiness is claimed.

## Sources and attribution

The measurements originate from **James Moore and Daniela Sawyer**, *High-Frequency Sensor Signals from a One-Year Long Accelerated Wear Test of a KUKA KR16 Industrial Robotic Arm*, Sheffield ORDA, [DOI: 10.15131/shef.data.27368490.v1](https://doi.org/10.15131/shef.data.27368490.v1).

Associated publication: Moore and Sawyer, *Equipment Health Monitoring for Industrial Robotic Arms*, IEEE CASE 2024, [DOI: 10.1109/CASE59546.2024.10711629](https://doi.org/10.1109/CASE59546.2024.10711629).

The original dataset and publication are external contributions, not publications by this repository's author. This personal project is not an official institutional publication or endorsement. Dataset permissions do not by themselves grant a licence to every item in this repository.

