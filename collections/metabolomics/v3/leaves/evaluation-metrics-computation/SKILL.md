---
name: evaluation-metrics-computation
description: Use when simulated Environment runs over the same chemical set with different
  controllers (e.g. TopNController, SmartROIController, WeightedDEWController) must
  be quantitatively compared on fragmentation coverage and intensity.
when_to_use_negative:
- Input is real instrument data or a finished acquisition file rather than a ViMMS
  simulated Environment run — the evaluation API assesses simulated controller behaviour
- Goal is to validate ViMMS itself on real hardware; the paper defers such validation
  to Davies et al. (2021) via the Thermo Fisher IAPI bridging code
- Only a single controller was run and no comparative benchmark is needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: ViMMS 2.0
  role: Framework whose evaluation methods compute coverage and intensities of fragmented
    precursors for benchmarking controllers
  repo: https://github.com/glasgowcompbio/vimms
- name: TopNController
  role: Baseline DDA controller whose simulated run supplies the evaluation input
  repo: https://github.com/glasgowcompbio/vimms
- name: SmartROIController
  role: Enhanced ROI-guided controller benchmarked against Top-N in the comparative
    evaluation
  repo: https://github.com/glasgowcompbio/vimms
- name: WeightedDEWController
  role: Dynamic-exclusion-weighted Top-N variant benchmarked in the comparative evaluation
  repo: https://github.com/glasgowcompbio/vimms
- name: Environment
  role: Runs mass spectrometer and controller; provides write_mzML to export generated
    scans for evaluation
  repo: https://github.com/glasgowcompbio/vimms
- name: pandas
  role: Aggregates per-controller evaluation reports into comparative_metrics.csv
- name: numpy
  role: Numeric support for intensity/coverage summary computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:b940e347e6b6584223b32df0a5fd2d075908a551de4f4bf650e57d778d6aceb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Environment
  - IndependentMassSpectrometer
  - SmartROIController
  - TopNController
  - ViMMS 2.0
  - WeightedDEWController
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# evaluation-metrics-computation

## Summary

Compute coverage and fragmented-precursor intensity metrics in ViMMS 2.0 via vimms.Evaluation, then aggregate per-controller results (e.g. Top-N, SmartROI, WeightedDEW) into a comparative table to benchmark DDA acquisition strategies in-silico.

## When to use

Use when simulated Environment runs over the same chemical set with different controllers (e.g. TopNController, SmartROIController, WeightedDEWController) must be quantitatively compared on fragmentation coverage and intensity.

## When NOT to use

- Input is real instrument data or a finished acquisition file rather than a ViMMS simulated Environment run — the evaluation API assesses simulated controller behaviour
- Goal is to validate ViMMS itself on real hardware; the paper defers such validation to Davies et al. (2021) via the Thermo Fisher IAPI bridging code
- Only a single controller was run and no comparative benchmark is needed

## Inputs

- ViMMS Environment objects from completed controller runs (TopNController, SmartROIController, WeightedDEWController) over IndependentMassSpectrometer simulations
- Chemical mixtures prepared via ChemicalMixtureCreator or extracted from existing mzML files (e.g. Beer_multibeers_1_T10_POS.mzML)
- Optionally exported mzML scan outputs from Environment.write_mzML

## Outputs

- Per-controller evaluation report of times each precursor was fragmented (times_fragmented_summary), cumulative coverage and intensity statistics
- Aggregated comparative table (comparative_metrics.csv) with columns: controller, chemical, times fragmented, coverage, intensity summary

## How to apply

After running each controller (TopNController, SmartROIController, WeightedDEWController) in ViMMS Environment instances over IndependentMassSpectrometer simulations of the same chemical mixtures (e.g. produced by ChemicalMixtureCreator), call the evaluation methods ViMMS provides to compute, per chemical precursor, how many times it was fragmented and its cumulative coverage and intensity statistics (times_fragmented_summary). Aggregate the per-controller evaluation reports into a single table with columns for controller, chemical, times fragmented, coverage and intensity summary, and export it as a CSV (e.g. comparative_metrics.csv). Validate that the table exists, parses as well-formed CSV, and contains evaluation rows for all three controllers, consistent with the framework's reported metrics. Note the paper reports no benchmark numbers itself; evaluation is executed on your own simulated runs.

## Related tools

- **ViMMS 2.0** (Framework whose evaluation methods compute coverage and intensities of fragmented precursors for benchmarking controllers) — https://github.com/glasgowcompbio/vimms
- **TopNController** (Baseline DDA controller whose simulated run supplies the evaluation input) — https://github.com/glasgowcompbio/vimms
- **SmartROIController** (Enhanced ROI-guided controller benchmarked against Top-N in the comparative evaluation) — https://github.com/glasgowcompbio/vimms
- **WeightedDEWController** (Dynamic-exclusion-weighted Top-N variant benchmarked in the comparative evaluation) — https://github.com/glasgowcompbio/vimms
- **Environment** (Runs mass spectrometer and controller; provides write_mzML to export generated scans for evaluation) — https://github.com/glasgowcompbio/vimms
- **pandas** (Aggregates per-controller evaluation reports into comparative_metrics.csv)
- **numpy** (Numeric support for intensity/coverage summary computation)

## Examples

```
from vimms.Evaluation import evaluate_simulated_env; report = evaluate_simulated_env(env)
```

## Evaluation signals

- comparative_metrics.csv exists and parses as well-formed CSV
- Evaluation rows present for all three controllers (Top-N, SmartROI, WeightedDEW) over the same chemical set
- times_fragmented values per chemical are consistent with the framework's times_fragmented_summary output
- Every chemical in the input mixture appears in the aggregated table (full coverage of the chemical set)
- Controller comparisons are made on identical simulated chemical mixtures (same upstream task_001 inputs)

## Limitations

- The paper itself reports no evaluation results, benchmarks, or quantitative performance data — metrics must be produced from your own simulated runs
- min_intensity threshold (1.75E5 in the example, to make notebooks complete quickly) trades simulation accuracy for speed; lowering to ~1E4 is more accurate but slower with higher memory use, and 0 includes all ROIs including noise
- Validation of ViMMS on real instruments is out of scope here and deferred to Davies et al. (2021)

## Evidence

- [methods] evaluation API: [evidence span withheld — anchor status: not_found]
- [full_text] task finding: [evidence span withheld — anchor status: not_found]
- [abstract] controllers as examples: "SmartROI and WeightedDEW
outlined by Davies et al. (2021)) are also provided that demonstrate how novel fragmentation
strategies could be rapidly implemented and validated in ViMMS 2.0"
- [methods] environment usage: "create an environment to run both the mass spec and controller"
- [results] min_intensity caveat: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [results] threshold 0 semantics: "Setting the threshold to 0 means using all ROIs as simulation input, including even the noise"
- [methods] mzML export: "The `Environment` class provides `write_mzML` to export the generated scans"
