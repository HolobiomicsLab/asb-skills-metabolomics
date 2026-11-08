---
name: dd-acquisition-strategy-simulation
description: Use when you need to prototype, compare or optimise MS/MS acquisition
  strategies in-silico — e.g. benchmarking Top-N against SmartROI/WeightedDEW controllers
  — before committing to runs on a real Thermo Tribrid Fusion instrument.
when_to_use_negative:
- You already have real instrument-acquired data and want to evaluate a method on
  actual spectra rather than simulate acquisition in ViMMS
- You need data-independent acquisition strategies such as All-ion-fragmentation or
  SWATH-MS rather than Top-N-style DDA controller comparison
- You require validated quantitative performance claims — this paper reports no benchmarks
  itself; controller utility validation on real instruments is deferred to Davies
  et al. (2021)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Simulation framework providing the Environment, IndependentMassSpectrometer,
    base Controller class and Top-N, SmartROI and WeightedDEW controllers
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which controllers are implemented and run (requires Python 3.0
    or higher)
- name: pandas
  role: Dependency used to aggregate per-controller evaluation reports into a comparative
    CSV table
- name: numpy
  role: Common dependency of ViMMS used in simulation and evaluation code
- name: Thermo Fisher IAPI
  role: Optional bridging layer to execute the same controllers unchanged on a Thermo
    Tribrid Fusion instrument after in-silico benchmarking
  repo: https://github.com/thermofisherlsms/iapi
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:936e91192b6c47a1a212d8c99e03c55f493ccbd2fccd4d2c4da1032298f54645
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - IndependentMassSpectrometer
  - Python
  - SmartROIController
  - Thermo Fisher IAPI
  - TopNController
  - ViMMS 2.0
  - WeightedDEWController
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# dd-acquisition-strategy-simulation

## Summary

Simulate and comparatively benchmark data-dependent acquisition (DDA) fragmentation strategies in ViMMS 2.0 — running Top-N, SmartROI and WeightedDEW controllers over the same simulated chemical mixture and evaluating coverage and intensity of fragmented precursors.

## When to use

Use when you need to prototype, compare or optimise MS/MS acquisition strategies in-silico — e.g. benchmarking Top-N against SmartROI/WeightedDEW controllers — before committing to runs on a real Thermo Tribrid Fusion instrument.

## When NOT to use

- You already have real instrument-acquired data and want to evaluate a method on actual spectra rather than simulate acquisition in ViMMS
- You need data-independent acquisition strategies such as All-ion-fragmentation or SWATH-MS rather than Top-N-style DDA controller comparison
- You require validated quantitative performance claims — this paper reports no benchmarks itself; controller utility validation on real instruments is deferred to Davies et al. (2021)

## Inputs

- Chemical mixture produced by ChemicalMixtureCreator (synthetic sampling from HMDB, e.g. hmdb_compounds.p, or extracted from mzML files such as Beer_multibeers_1_T10_POS.mzML)
- Controller configurations for TopNController, SmartROIController and WeightedDEWController
- min_intensity threshold setting for ROI-based simulation input (e.g. 1.75E5, 1E4, or 0)

## Outputs

- Simulated scan results exportable as mzML via Environment.write_mzML
- Per-controller evaluation metrics: times each chemical precursor was fragmented, coverage and intensity statistics (times_fragmented_summary)
- Aggregated comparative table (e.g. comparative_metrics.csv) with columns for controller, chemical, times fragmented, coverage and intensity summary

## How to apply

Create a chemical dataset (synthetically via ChemicalMixtureCreator or extracted from mzML such as Beer_multibeers_1_T10_POS.mzML), then instantiate TopNController, SmartROIController and WeightedDEWController — each extending ViMMS's base Controller class. Run each controller independently inside an Environment together with an IndependentMassSpectrometer over the same chemical set so the comparison is controlled. After each run, use ViMMS's evaluation methods to compute, per chemical, how many times each precursor was fragmented and its cumulative coverage and intensity statistics. Aggregate per-controller evaluation reports into a single table (controller, chemical, times fragmented, coverage, intensity summary) exported as CSV. Note the min_intensity threshold trade-off: 1.75E5 completes quickly, 1E4 is more accurate but slower, 0 includes all ROIs including noise.

## Related tools

- **ViMMS 2.0** (Simulation framework providing the Environment, IndependentMassSpectrometer, base Controller class and Top-N, SmartROI and WeightedDEW controllers) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which controllers are implemented and run (requires Python 3.0 or higher))
- **pandas** (Dependency used to aggregate per-controller evaluation reports into a comparative CSV table)
- **numpy** (Common dependency of ViMMS used in simulation and evaluation code)
- **Thermo Fisher IAPI** (Optional bridging layer to execute the same controllers unchanged on a Thermo Tribrid Fusion instrument after in-silico benchmarking) — https://github.com/thermofisherlsms/iapi

## Examples

```
from vimms.Controller import TopNController, SmartROIController, WeightedDEWController; env = Environment(MassSpec, controller); env.run(); write_mzML()
```

## Evaluation signals

- comparative_metrics.csv exists, parses as well-formed CSV, and contains evaluation rows for all three controllers (Top-N, SmartROI, WeightedDEW)
- Per-chemical times_fragmented_summary values in the aggregated table are consistent with the framework's own reported evaluation metrics
- All three controllers ran over the identical chemical mixture, so differences in coverage/intensity reflect acquisition strategy, not input differences
- Environment runs completed and mzML output could be written via Environment.write_mzML for each controller run

## Limitations

- The min_intensity threshold trades speed for fidelity: 1.75E5 is deliberately high so notebooks complete quickly, 1E4 is more accurate but much slower with higher memory usage, and 0 includes all ROIs including noise
- This paper itself reports no evaluation results, benchmarks or quantitative performance data for the controllers; validation on real instruments appears in a separate publication (Davies et al., 2021)
- No explicit software version number is stated for ViMMS 2.0 in the paper, and no changelog was found for the repository

## Evidence

- [abstract] SmartROI and WeightedDEW example enhanced DDA controllers: "SmartROI and WeightedDEW
outlined by Davies et al. (2021)) are also provided that demonstrate how novel fragmentation
strategies could be rapidly implemented and validated in ViMMS 2.0"
- [abstract] controllers extend base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [methods] create Environment to run mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] evaluation methods exist: "ViMMS allows evaluation of simulation results using methods provided [here]"
- [results] min_intensity threshold trade-off: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [readme] README framing of ViMMS purpose: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics"
