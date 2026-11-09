---
name: in-silico-dda-controller-grid-search-optimization
description: Use when DDA controller parameters (e.g., SmartROI alpha/beta or WeightedDEW
  t0/t1) must be tuned before hardware experiments and full-scan mzML seed data exist
  for the samples of interest.
when_to_use_negative:
- When the goal is to optimize parameters for a controller other than SmartROI or
  WeightedDEW, or when the sample's full-scan mzML seed data are unavailable for ViMMS
- When real-instrument validation data already exist and the grid can be evaluated
  directly on hardware without simulation
- When the objective is not coverage of fragmented unique ions (e.g., optimizing for
  intensity or reproducibility metrics not modeled in the simulation)
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to seed full-scan mzML files and
    simulate the 30 (SmartROI) and 36 (WeightedDEW) virtual injections per extract
    during the grid search
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:fff5cfaf6d4bc14d58734b5defa4fddb79ead00017b9dbaedc087c78524e982c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViMMS
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# in-silico-dda-controller-grid-search-optimization

## Summary

Uses the ViMMS virtual metabolomics mass spectrometer to grid-search DDA controller parameters (SmartROI alpha/beta, WeightedDEW t0/t1) on full-scan mzML-seeded simulations, selecting settings that maximize coverage before any real instrument time is spent.

## When to use

Use when DDA controller parameters (e.g., SmartROI alpha/beta or WeightedDEW t0/t1) must be tuned before hardware experiments and full-scan mzML seed data exist for the samples of interest.

## When NOT to use

- When the goal is to optimize parameters for a controller other than SmartROI or WeightedDEW, or when the sample's full-scan mzML seed data are unavailable for ViMMS
- When real-instrument validation data already exist and the grid can be evaluated directly on hardware without simulation
- When the objective is not coverage of fragmented unique ions (e.g., optimizing for intensity or reproducibility metrics not modeled in the simulation)

## Inputs

- Full-scan mzML files for serum (QCA) and beer (QCB) extracts used to seed ViMMS
- Controller parameter grids: SmartROI (alpha, beta) and WeightedDEW (t0, t1)

## Outputs

- Selected SmartROI parameters (alpha = 1000, beta = 0.1)
- Selected WeightedDEW parameters (t0 = 15 s, t1 = 120 s)
- Tabulated controller_optimization_results.csv with selected parameters and virtual-injection counts per controller and extract

## How to apply

Seed ViMMS with the full-scan mzML files for each extract (here serum QCA and beer QCB) so the virtual metabolomics mass spectrometer reproduces realistic ion elution. Grid-search the SmartROI controller over its alpha (refragmentation intensity-increase factor) and beta (drop-factor from highest intensity since last fragmentation) space, running 30 virtual injections per extract (about 1 h of simulation per injection), maximizing coverage. Separately grid-search the WeightedDEW controller over t0 (hard exclusion duration) and t1 (time at which the linear weight reaches 1), requiring 36 virtual injections per extract. For each controller and extract, select the parameter combination with the highest simulated coverage; the validated selections are alpha = 1000 and beta = 0.1 for SmartROI and t0 = 15 s and t1 = 120 s for WeightedDEW, chosen because they performed well for both data sets. Tabulate selected parameters per controller together with the virtual-injection counts (30 for SmartROI, 36 for WeightedDEW) so the optimization is auditable.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to seed full-scan mzML files and simulate the 30 (SmartROI) and 36 (WeightedDEW) virtual injections per extract during the grid search)

## Evaluation signals

- Recovered SmartROI parameters exactly match alpha = 1000 and beta = 0.1 and WeightedDEW parameters match t0 = 15 s and t1 = 120 s
- Search consumed 30 virtual injections per extract for SmartROI and 36 per extract for WeightedDEW
- Each virtual injection takes about 1 h in simulation, so total runtime is roughly 30 h (SmartROI) and 36 h (WeightedDEW) per extract
- For each controller and extract, the chosen combination is the one with the highest simulated coverage among tested grid points
- The selected settings perform well for both data sets (serum QCA and beer QCB), not just one

## Limitations

- The grid search is computationally heavy: each virtual injection takes about 1 h in simulation and 30–36 injections are needed per extract per controller
- Optimal parameters were selected on two specific complex mixtures (serum and beer extracts); transferability to other sample types is not established
- Selection is based on maximizing simulated coverage; performance on real hardware must be confirmed separately

## Evidence

- [abstract] using a previously developed virtual metabolomics mass spectrometer (ViMMS): "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [intro] we describe how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS),10 reducing: "we describe how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS),10"
