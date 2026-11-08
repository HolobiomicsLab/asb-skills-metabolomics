---
name: weighted-dew-exclusion-window-parameter-tuning
description: 'Use when tuning dynamic-exclusion behaviour of a DDA method in ViMMS:
  full-scan mzML files of complex mixtures (e.g. serum/beer extracts) are available
  and t0/t1 values for WeightedDEW must be chosen by maximizing simulated coverage
  before instrument validation.'
when_to_use_negative:
- When optimizing the SmartROI controller instead — its grid search sweeps alpha (refragmentation
  intensity-increase factor) and beta (drop-factor from highest intensity since last
  fragmentation) with 30 virtual injections per extract, not t0/t1 with 36.
- When the goal is to evaluate acquisition strategies on real hardware via IAPI rather
  than to select exclusion-window parameters in silico.
- When input is MS2-level data or already-acquired fragmentation spectra — ViMMS needs
  full-scan (MS1 survey) mzML files to simulate DDA decisions.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to seed the simulator with full-scan
    mzML files and run the WeightedDEW controller grid search in silico, scoring each
    (t0, t1) combination by simulated coverage over 36 virtual injections per extract
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:80fa9389d11c6fd26c53cb033c1f5fd40cc6417357e910f6a349a42b93797a63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViMMS
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# weighted-dew-exclusion-window-parameter-tuning

## Summary

In silico grid-search tuning of the WeightedDEW DDA exclusion controller in ViMMS, sweeping the hard-exclusion duration (t0) and linear-weight ramp time (t1) to maximize MS2 coverage on virtual injections of complex extracts. The published search selected t0 = 15 s and t1 = 120 s as the settings that performed well on both serum and beer extracts.

## When to use

Use when tuning dynamic-exclusion behaviour of a DDA method in ViMMS: full-scan mzML files of complex mixtures (e.g. serum/beer extracts) are available and t0/t1 values for WeightedDEW must be chosen by maximizing simulated coverage before instrument validation.

## When NOT to use

- When optimizing the SmartROI controller instead — its grid search sweeps alpha (refragmentation intensity-increase factor) and beta (drop-factor from highest intensity since last fragmentation) with 30 virtual injections per extract, not t0/t1 with 36.
- When the goal is to evaluate acquisition strategies on real hardware via IAPI rather than to select exclusion-window parameters in silico.
- When input is MS2-level data or already-acquired fragmentation spectra — ViMMS needs full-scan (MS1 survey) mzML files to simulate DDA decisions.

## Inputs

- Full-scan mzML files for the serum (QCA) and beer (QCB) extracts
- WeightedDEW controller parameter grid over t0 (hard exclusion duration) and t1 (linear-weight ramp time)
- Virtual metabolomics mass spectrometer (ViMMS) environment seeded with the mzML files

## Outputs

- Selected WeightedDEW parameter settings per extract (published optimum: t0 = 15 s, t1 = 120 s)
- Simulated coverage values for each (t0, t1) grid point (36 virtual injections per extract)
- Tabulated controller-optimization results (e.g. controller_optimization_results.csv) with parameters and virtual-injection counts

## How to apply

Load the full-scan mzML files for the serum (QCA) and beer (QCB) extracts into ViMMS and use them to seed the virtual metabolomics mass spectrometer. Run a grid search of the WeightedDEW controller over the t0 (hard exclusion duration) and t1 (time at which the linear weight reaches 1) parameter space, with the objective of maximizing coverage — the proportion of ions fragmented. The published search required 36 virtual injections per extract, with each sample taking about 1 h in simulation. For each extract, select the (t0, t1) combination with the highest simulated coverage; the paper's grid search yielded t0 = 15 s and t1 = 120 s as settings that performed well on both data sets. Tabulate the selected parameters per controller (alongside the SmartROI grid-search results: alpha = 1000, beta = 0.1) and confirm the recovered values match the reported selections and injection counts.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to seed the simulator with full-scan mzML files and run the WeightedDEW controller grid search in silico, scoring each (t0, t1) combination by simulated coverage over 36 virtual injections per extract)

## Examples

```
from vimms.Controller import WeightedDEW; c = WeightedDEW(mzml, t0=15, t1=120)  # paper-selected optimum over the t0/t1 coverage grid
```

## Evaluation signals

- Recovered optimum matches the reported selection: t0 = 15 s and t1 = 120 s for WeightedDEW.
- The search consumed 36 virtual injections per extract, as reported for the WeightedDEW grid search (30 for the parallel SmartROI search).
- The selected (t0, t1) combination is the one with the highest simulated coverage for each of the serum (QCA) and beer (QCB) extracts.
- The output table records both the selected parameters and the virtual-injection counts per controller, consistent with the published tabulation of controller-optimization results.
- Coverage objective is consistent with the paper's framing of improving DDA coverage — the proportion of ions fragmented.

## Limitations

- Tuning is performed in silico with ViMMS; each simulated injection takes about 1 h per sample, so grid searches are compute-intensive (36 virtual injections per extract for WeightedDEW).
- Parameters are selected to perform well on the two test data sets (serum QCA and beer QCB); coverage on other sample matrices may differ.
- The in silico optimum still requires validation on real samples via IAPI before deployment on actual hardware.

## Evidence

- [methods] WeightedDEW grid-search scope and objective: [evidence span withheld — anchor status: not_found]
- [methods] Selected parameters and injection count: [evidence span withheld — anchor status: not_found]
- [results] Published grid-search optimum: [evidence span withheld — anchor status: not_found]
- [abstract] In silico prototyping rationale: "development and optimization of two new DDA methods that introduce new advanced ion prioritization strategies."
- [intro] Simulation-to-hardware validation path: "new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS)"
