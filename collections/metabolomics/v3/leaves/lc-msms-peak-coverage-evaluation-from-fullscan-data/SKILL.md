---
name: lc-msms-peak-coverage-evaluation-from-fullscan-data
description: Use when the goal is to quantify DDA coverage (proportion of MS1 ions
  fragmented) for SmartROI or WeightedDEW controller settings by simulating LC-MS/MS
  injections from full-scan mzML data in ViMMS.
when_to_use_negative:
- The output required is validated performance on real hardware rather than in silico
  ranking — the article validates controllers on real samples via IAPI instead of
  ViMMS
- The input is already a fragmented DDA run with measured coverage, so no virtual
  reinjection simulation is needed
- The goal is fragmenting a small subset of high-intensity ions rather than improving
  single-injection DDA coverage
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to seed simulations from full-scan
    mzML and run SmartROI/WeightedDEW controller grid searches to measure coverage
    in silico
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:3b5d4b9f6b6d445e2784ca711e7257b2fee517e3b2919ff4d0e77afe9e68ad16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViMMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# LC-MS/MS peak coverage evaluation from full-scan data

## Summary

Evaluates DDA controller peak coverage in silico by seeding the ViMMS virtual metabolomics mass spectrometer with full-scan mzML files and measuring the proportion of ions fragmented under different controller parameter settings.

## When to use

Use when the goal is to quantify DDA coverage (proportion of MS1 ions fragmented) for SmartROI or WeightedDEW controller settings by simulating LC-MS/MS injections from full-scan mzML data in ViMMS.

## When NOT to use

- The output required is validated performance on real hardware rather than in silico ranking — the article validates controllers on real samples via IAPI instead of ViMMS
- The input is already a fragmented DDA run with measured coverage, so no virtual reinjection simulation is needed
- The goal is fragmenting a small subset of high-intensity ions rather than improving single-injection DDA coverage

## Inputs

- Full-scan (MS1) mzML files for the complex samples to be simulated (e.g., serum QCA and beer QCB extracts)
- Controller parameter grids (SmartROI alpha/beta; WeightedDEW t0/t1)

## Outputs

- Simulated coverage values per controller, parameter combination, and extract
- Selected optimal controller parameters (SmartROI alpha=1000, beta=0.1; WeightedDEW t0=15 s, t1=120 s)
- controller_optimization_results.csv tabulating parameters and virtual-injection counts (30 for SmartROI, 36 for WeightedDEW)

## How to apply

Load the full-scan mzML files for the extracts (e.g., serum QCA and beer QCB) into ViMMS and use them to seed the virtual metabolomics mass spectrometer. Run a grid search of the SmartROI controller over the alpha (refragmentation intensity-increase factor) and beta (drop-factor from highest intensity since last fragmentation) parameter space, maximizing coverage, requiring 30 virtual injections per extract, with each simulated sample taking about 1 h. Run a grid search of the WeightedDEW controller over t0 (hard exclusion duration) and t1 (time at which the linear weight reaches 1), requiring 36 virtual injections per extract. For each controller and extract, select the parameter combination with the highest simulated coverage — the reported selections are alpha = 1000 and beta = 0.1 for SmartROI, and t0 = 15 s and t1 = 120 s for WeightedDEW. Tabulate the selected parameters and virtual-injection counts and write them to controller_optimization_results.csv.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to seed simulations from full-scan mzML and run SmartROI/WeightedDEW controller grid searches to measure coverage in silico)

## Evaluation signals

- Recovered optimal parameters match the reported selections: SmartROI alpha = 1000, beta = 0.1; WeightedDEW t0 = 15 s, t1 = 120 s
- Grid search consumed the expected virtual injections per extract: 30 for SmartROI and 36 for WeightedDEW
- Coverage is computed against the ions present in the full-scan mzML seed files for both QCA (serum) and QCB (beer) extracts
- controller_optimization_results.csv is written with per-controller parameter selections and injection counts

## Limitations

- Each simulated injection takes about 1 h, so full grid searches require many hours of compute per extract
- Results are in silico; controller performance must still be validated on real samples via the Instrument Application Programming Interface (IAPI)
- Selections are specific to the simulated data sets (serum and beer extracts) and may not transfer to other matrices

## Evidence

- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] using a previously developed virtual metabolomics mass spectrometer (ViMMS): "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
- [methods] Run a grid search of the SmartROI controller over the alpha (refragmentation intensity-increase factor) and beta (drop-factor from highest intensity since last fragmentation) parameter space, maximizing coverage: [evidence span withheld — anchor status: not_found]
- [results] select the parameter combination with the highest simulated coverage, yielding alpha = 1000 and beta = 0.1 for SmartROI and t0 = 15 s and t1 = 120 s for WeightedDEW: [evidence span withheld — anchor status: not_found]
