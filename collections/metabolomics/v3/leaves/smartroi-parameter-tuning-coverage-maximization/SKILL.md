---
name: smartroi-parameter-tuning-coverage-maximization
description: Use when developing or tuning DDA acquisition-controller parameters in
  silico before spending instrument time — specifically when maximizing fragmentation
  coverage of ions in complex metabolite mixtures via ViMMS virtual injections.
when_to_use_negative:
- When validating the optimized controllers on real hardware rather than in silico
  — that step uses the Instrument Application Programming Interface (IAPI), not the
  ViMMS grid search.
- When the goal is a single-injection theoretical optimum rather than practical controller
  tuning — the article computes theoretically optimal DDA performance separately from
  controller parameter selection.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to run grid searches of the SmartROI
    and WeightedDEW controllers over the alpha/beta and t0/t1 parameter spaces, maximizing
    simulated coverage per virtual injection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:389202414220ca0d3a3662ff5fb55caba734f196f364182e1fe9996f044e5444
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViMMS
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# smartroi-parameter-tuning-coverage-maximization

## Summary

Grid-search optimization of SmartROI (alpha, beta) and WeightedDEW (t0, t1) controller parameters in ViMMS to maximize simulated DDA fragmentation coverage, selecting the settings (alpha=1000, beta=0.1; t0=15 s, t1=120 s) that performed best on both serum and beer extracts.

## When to use

Use when developing or tuning DDA acquisition-controller parameters in silico before spending instrument time — specifically when maximizing fragmentation coverage of ions in complex metabolite mixtures via ViMMS virtual injections.

## When NOT to use

- When validating the optimized controllers on real hardware rather than in silico — that step uses the Instrument Application Programming Interface (IAPI), not the ViMMS grid search.
- When the goal is a single-injection theoretical optimum rather than practical controller tuning — the article computes theoretically optimal DDA performance separately from controller parameter selection.

## Inputs

- Full-scan mzML files for the serum (QCA) extract
- Full-scan mzML files for the beer (QCB) extract
- SmartROI controller parameter grid (alpha refragmentation intensity-increase factor, beta drop-factor from highest intensity since last fragmentation)
- WeightedDEW controller parameter grid (t0 hard exclusion duration, t1 time at which the linear weight reaches 1)

## Outputs

- Selected SmartROI parameters (alpha = 1000, beta = 0.1)
- Selected WeightedDEW parameters (t0 = 15 s, t1 = 120 s)
- controller_optimization_results.csv tabulating selected parameters per controller and virtual-injection counts (30 for SmartROI, 36 for WeightedDEW)

## How to apply

Load full-scan mzML files for the serum (QCA) and beer (QCB) extracts into ViMMS and use them to seed the virtual metabolomics mass spectrometer. Run a grid search of the SmartROI controller over the alpha (refragmentation intensity-increase factor) and beta (drop-factor from highest intensity since last fragmentation) parameter space, maximizing coverage; this required 30 virtual injections per extract, each taking about 1 h in simulation. Run a second grid search of the WeightedDEW controller over t0 (hard exclusion duration) and t1 (time at which the linear weight reaches 1), requiring 36 virtual injections per extract. For each controller and extract, select the parameter combination with the highest simulated coverage, which yielded alpha=1000 and beta=0.1 for SmartROI and t0=15 s and t1=120 s for WeightedDEW — the settings that performed well for both data sets. Tabulate the selected parameters per controller together with the virtual-injection counts and write the results to controller_optimization_results.csv.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to run grid searches of the SmartROI and WeightedDEW controllers over the alpha/beta and t0/t1 parameter spaces, maximizing simulated coverage per virtual injection)

## Examples

```
vimms simulate --controller SmartROI --alpha 1000 --beta 0.1 --injections 30 --input QCA.mzML QCB.mzML; vimms simulate --controller WeightedDEW --t0 15 --t1 120 --injections 36 --input QCA.mzML QCB.mzML
```

## Evaluation signals

- Recovered SmartROI parameters match the reported selections: alpha = 1000 and beta = 0.1
- Recovered WeightedDEW parameters match the reported selections: t0 = 15 s and t1 = 120 s
- Grid-search consumed 30 virtual injections per extract for SmartROI and 36 virtual injections per extract for WeightedDEW
- Selected parameter combination is the one with the highest simulated coverage for each controller on both the serum (QCA) and beer (QCB) extracts
- controller_optimization_results.csv contains the per-controller selected parameters and virtual-injection counts

## Limitations

- Each simulated injection takes about 1 h in simulation, so the grid searches are costly (30 SmartROI and 36 WeightedDEW virtual injections per extract).
- Selected parameters must perform well across both data sets (serum and beer), which may sacrifice extract-specific optimality.

## Evidence

- [methods] grid search over SmartROI parameters: [evidence span withheld — anchor status: not_found]
- [methods] grid search over WeightedDEW parameters: [evidence span withheld — anchor status: not_found]
- [results] selected parameters: [evidence span withheld — anchor status: not_found]
- [intro] ViMMS as the framework: "we describe how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS)"
