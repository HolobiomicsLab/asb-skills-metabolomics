---
name: vimms-virtual-mass-spectrometer-simulation
description: Use when you need to prototype, optimize, or compare DDA acquisition-controller
  parameter settings in silico (e.g. SmartROI alpha/beta, WeightedDEW t0/t1) from
  full-scan mzML files, avoiding costly trial-and-error LC-MS/MS injections on real
  hardware.
when_to_use_negative:
- Input is already-acquired MS2/fragmentation data intended for metabolite identification
  or annotation rather than acquisition-method design
- The goal is real-instrument acquisition itself — in silico grid-search selections
  must still be validated on real samples (via the IAPI interface) before being trusted
- You need results in minutes and cannot afford ~1 h of simulation per virtual injection
  times dozens of parameter combinations
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer seeded from full-scan mzML files; runs
    simulated DDA controllers (SmartROI, WeightedDEW) over parameter grids to measure
    simulated fragmentation coverage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:025b2cb4355dc98b0dae6f168c0b1eedf8f0690da28e83c76ff13e8c3d8f552b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViMMS
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# vimms-virtual-mass-spectrometer-simulation

## Summary

Use ViMMS, a virtual metabolomics mass spectrometer, to run in silico grid searches over DDA controller parameters (SmartROI, WeightedDEW) on full-scan mzML data, selecting the settings that maximize fragmentation coverage before any real-instrument time is spent.

## When to use

Use when you need to prototype, optimize, or compare DDA acquisition-controller parameter settings in silico (e.g. SmartROI alpha/beta, WeightedDEW t0/t1) from full-scan mzML files, avoiding costly trial-and-error LC-MS/MS injections on real hardware.

## When NOT to use

- Input is already-acquired MS2/fragmentation data intended for metabolite identification or annotation rather than acquisition-method design
- The goal is real-instrument acquisition itself — in silico grid-search selections must still be validated on real samples (via the IAPI interface) before being trusted
- You need results in minutes and cannot afford ~1 h of simulation per virtual injection times dozens of parameter combinations

## Inputs

- Full-scan (MS1) mzML files for the samples to be simulated (e.g. serum QCA and beer QCB extracts)
- Controller specification and parameter grid (SmartROI alpha/beta ranges; WeightedDEW t0/t1 ranges)
- Number of virtual injections per parameter combination (30 for SmartROI, 36 for WeightedDEW in this study)

## Outputs

- Selected controller parameter settings per controller and extract (SmartROI alpha=1000, beta=0.1; WeightedDEW t0=15 s, t1=120 s)
- Simulated fragmentation coverage per parameter combination
- Tabulated results, e.g. controller_optimization_results.csv with parameters and virtual-injection counts

## How to apply

Load the full-scan mzML files for the target extracts (in this study, serum QCA and beer QCB) and use them to seed the ViMMS virtual metabolomics mass spectrometer. Run a grid search of the SmartROI controller over the alpha (refragmentation intensity-increase factor) and beta (drop-factor from highest intensity since last fragmentation) parameter space, maximizing coverage — this required 30 virtual injections per extract, each taking about 1 h in simulation. Separately, run a grid search of the WeightedDEW controller over t0 (hard exclusion duration) and t1 (time at which the linear weight reaches 1), maximizing coverage with 36 virtual injections per extract. For each controller and extract, select the parameter combination with the highest simulated coverage; for this study that yielded alpha = 1000 and beta = 0.1 for SmartROI, and t0 = 15 s and t1 = 120 s for WeightedDEW. The rationale is that DDA typically only fragments a small subset of ions, so in silico optimization identifies ion-prioritization settings that fragment more unique ions than standard Top-N DDA before validating on real samples.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer seeded from full-scan mzML files; runs simulated DDA controllers (SmartROI, WeightedDEW) over parameter grids to measure simulated fragmentation coverage)

## Evaluation signals

- Recovered SmartROI parameters match the reported selections: alpha = 1000 and beta = 0.1
- Recovered WeightedDEW parameters match the reported selections: t0 = 15 s and t1 = 120 s
- Grid search consumed the expected virtual injections per extract: 30 for SmartROI and 36 for WeightedDEW
- Selected parameter combination corresponds to the highest simulated coverage for each controller and extract
- Tabulated output (e.g. controller_optimization_results.csv) contains both the selected parameters and the virtual-injection counts

## Limitations

- Simulation is compute-heavy: each virtual injection takes about 1 h, and the grid searches required 30 (SmartROI) and 36 (WeightedDEW) virtual injections per extract
- Optimal parameters were selected for two specific complex mixtures (serum QCA and beer QCB); settings may not transfer to other matrices
- In silico selections require subsequent validation on real samples before acquisition strategies are adopted

## Evidence

- [abstract] using a previously developed virtual metabolomics mass spectrometer (ViMMS): "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [intro] optimized and validated using ViMMS: "Second, we describe how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS)"
- [results] grid search selected parameters for both data sets: [evidence span withheld — anchor status: not_found]
