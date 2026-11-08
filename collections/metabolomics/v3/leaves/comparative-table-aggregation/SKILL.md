---
name: comparative-table-aggregation
description: Use when simulated DDA runs for multiple fragmentation controllers (e.g.
  Top-N, SmartROI, WeightedDEW) have completed in ViMMS 2.0 over the same chemical
  set and their evaluation reports must be merged into one comparative benchmark table.
when_to_use_negative:
- Only a single controller run exists and no cross-controller comparison is intended
  — aggregate-table assembly is unnecessary.
- Input is raw mzML scan output that has not yet been evaluated per-chemical (times
  fragmented / coverage / intensity) — evaluation must precede aggregation.
- The goal is running controllers unchanged on a real Thermo Tribrid Fusion instrument
  via Thermo Fisher IAPI bridging rather than in-silico benchmarking.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: ViMMS 2.0 (vimms)
  role: Provides Environment, the TopN/SmartROI/WeightedDEW controllers, IndependentMassSpectrometer
    simulation and the evaluation methods whose reports are aggregated
  repo: https://github.com/glasgowcompbio/vimms
- name: pandas
  role: Loads the per-controller evaluation reports, concatenates them with a controller
    column, and writes the aggregated comparative_metrics.csv
- name: numpy
  role: Supports numeric summarisation (e.g. cumulative coverage and intensity statistics)
    over per-chemical evaluation values
- name: TopNController / SmartROIController / WeightedDEWController
  role: The compared fragmentation strategies; each is run independently over the
    same chemical set to produce one evaluation report per controller
  repo: https://github.com/glasgowcompbio/vimms
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:1217991384feef9e6ee15f29347550572f15350033a836a20701bd12d8376d11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - IndependentMassSpectrometer
  - SmartROIController
  - TopNController
  - TopNController / SmartROIController / WeightedDEWController
  - ViMMS 2.0 (vimms)
  - WeightedDEWController
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# comparative-table-aggregation

## Summary

Aggregates per-controller ViMMS 2.0 simulation evaluation reports (times fragmented, coverage, intensity statistics per chemical precursor) for Top-N, SmartROI and WeightedDEW DDA controllers into a single comparative CSV table for benchmarking.

## When to use

Use when simulated DDA runs for multiple fragmentation controllers (e.g. Top-N, SmartROI, WeightedDEW) have completed in ViMMS 2.0 over the same chemical set and their evaluation reports must be merged into one comparative benchmark table.

## When NOT to use

- Only a single controller run exists and no cross-controller comparison is intended — aggregate-table assembly is unnecessary.
- Input is raw mzML scan output that has not yet been evaluated per-chemical (times fragmented / coverage / intensity) — evaluation must precede aggregation.
- The goal is running controllers unchanged on a real Thermo Tribrid Fusion instrument via Thermo Fisher IAPI bridging rather than in-silico benchmarking.

## Inputs

- Chemical mixture (e.g. from ChemicalMixtureCreator or extracted from an mzML file like Beer_multibeers_1_T10_POS.mzML)
- Per-controller ViMMS Environment simulation runs (TopNController, SmartROIController, WeightedDEWController over IndependentMassSpectrometer)
- Per-controller evaluation reports (times_fragmented_summary, coverage and intensity statistics per chemical)

## Outputs

- comparative_metrics.csv with columns: controller, chemical, times fragmented, coverage, intensity summary

## How to apply

Run TopNController, SmartROIController and WeightedDEWController independently in ViMMS Environment instances over IndependentMassSpectrometer simulations of the same chemical set (e.g. produced via ChemicalMixtureCreator or extracted from an mzML file such as Beer_multibeers_1_T10_POS.mzML). For each controller run, evaluate the simulated environment using ViMMS's evaluation methods to compute, per chemical precursor, the number of times it was fragmented, its cumulative coverage, and intensity summary statistics. Load each per-controller evaluation report with pandas, add a controller identifier column, and concatenate the reports row-wise. Write the aggregated frame to comparative_metrics.csv. Decide upfront whether all ROI-level input is simulated (threshold 0 includes noise) or a high min_intensity (e.g. 1.75E5) is used for speed, and keep the setting identical across controllers so the comparison is fair.

## Related tools

- **ViMMS 2.0 (vimms)** (Provides Environment, the TopN/SmartROI/WeightedDEW controllers, IndependentMassSpectrometer simulation and the evaluation methods whose reports are aggregated) — https://github.com/glasgowcompbio/vimms
- **pandas** (Loads the per-controller evaluation reports, concatenates them with a controller column, and writes the aggregated comparative_metrics.csv)
- **numpy** (Supports numeric summarisation (e.g. cumulative coverage and intensity statistics) over per-chemical evaluation values)
- **TopNController / SmartROIController / WeightedDEWController** (The compared fragmentation strategies; each is run independently over the same chemical set to produce one evaluation report per controller) — https://github.com/glasgowcompbio/vimms

## Examples

```
from vimms.Controller import TopNController, SmartROIController, WeightedDEWController; from vimms.Evaluation import evaluate_simulated_env; report = evaluate_simulated_env(env)
```

## Evaluation signals

- comparative_metrics.csv exists and parses as well-formed CSV with the expected header (controller, chemical, times fragmented, coverage, intensity summary).
- Evaluation rows are present for all three controllers (Top-N, SmartROI, WeightedDEW), i.e. every chemical appears once per controller.
- Per-chemical times-fragmented and coverage values in the aggregated table match the framework's reported per-controller evaluation metrics (times_fragmented_summary).
- Controller identifier column distinguishes exactly the three controller runs; no duplicate chemical-controller pairs.
- Coverage and intensity summary values are numerically consistent with the per-controller reports before concatenation (spot-check a few chemicals against the raw reports).

## Limitations

- The example min_intensity threshold (1.75E5) is set high only so notebooks complete quickly; lowering it to e.g. 1E4 gives a more accurate simulation but slower runs and higher memory usage, and threshold 0 includes all ROIs including noise — settings must be held constant across compared controllers.
- The JOSS paper reports no quantitative benchmark results itself; validation of ViMMS 2.0's utility on real instruments is deferred to Davies et al. (2021).
- Comparisons are only meaningful when all controllers are simulated over the identical chemical set and mass spectrometer configuration.

## Evidence

- [methods] Evaluation of simulated environment: ViMMS allows evaluation of simulation results using methods provided: "ViMMS allows evaluation of simulation results using methods provided [here]"
- [methods] Create an environment to run both mass spec and controller: "create an environment to run both the mass spec and controller"
- [abstract] SmartROI and WeightedDEW as example enhanced DDA controllers: "SmartROI and WeightedDEW
outlined by Davies et al. (2021)) are also provided that demonstrate how novel fragmentation
strategies could be rapidly implemented and validated in ViMMS 2.0"
- [results] min_intensity threshold trade-off for simulation speed vs accuracy: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [readme] Demo notebooks show how to use ViMMS in a simulated environment: "contains notebooks that demonstrate how to use the framework in a simulated environment"
