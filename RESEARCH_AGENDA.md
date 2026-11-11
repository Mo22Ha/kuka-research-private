# Research agenda: dynamics, inference and justified confidence

**Proposed work, not completed findings.** This independent investigation asks whether physical robot dynamics and statistical time-series models can jointly support a reliable Healthy-state reference.

The existing KUKA reports document conditional models, limited predictive gains and failed alarm calibration on later Healthy recordings. Better prediction alone neither identifies a physical mechanism nor validates a maintenance decision.

| Question | Concrete contribution | Evidence required |
|---|---|---|
| Which parameter combinations are identifiable? | Derive observation symmetries, sensitivities and profile likelihoods | Separate structural ambiguity from finite-data uncertainty; test on known synthetic truth |
| Is the sensor observation model adequate? | Specify body assignment, coordinate transformation, bandwidth and synchronization | Independent mounting documentation, joint-angle measurements and timebase checks |
| Can uncertainty transfer across days? | Compare day-level statistical references and model-discrepancy hypotheses | Chronological training/calibration/test blocks; interval coverage and false alarms on genuinely new days |
| When does prediction support explanation? | Compare observationally equivalent mechanisms and their distinguishing experiments | State what observations would falsify each hypothesis; avoid treating fit quality as proof |

## Small reproducible entry point

The [separate synthetic oscillator demonstration](https://github.com/Mo22Ha/physics-informed-modeling-under-uncertainty) explores amplitude–orientation ambiguity and forecast degradation after an observation-model change. It does not reuse KUKA implementation code or add empirical robot evidence. Its model-family advantage and restricted baseline are explicit.

## Proposed next experiment

Record a synchronized Healthy reference with independently measured angles, documented mounting orientation, calibrated measurement-chain response and repeated operating conditions across days. Define scoring windows and state initialization before evaluation. Freeze parameters and alarm rules before new condition-labelled data are inspected.

Bayesian inference, hierarchical models and conformal methods are candidate methods, not established results or automatic coverage guarantees. Temporal dependence and distribution shift require explicit assumptions and separate evaluation.

See [CONTRIBUTING.md](CONTRIBUTING.md) for a practical review format. Contact: [belboualia.mounir@tha.de](mailto:belboualia.mounir@tha.de).
