---
name: identification-rate-curve-analysis
description: Use when you need to characterize how simulated dataset complexity (number
  of peptides in a synthetic LC-MS/MS run) affects downstream peptide identification
  rates with a search-and-rescore pipeline.
when_to_use_negative:
- Your input is real experimental mzML data with unknown ground truth — SMITER is
  for creating synthetic gold-standard mzMLs from chemical formulas.
- You need nucleoside or lipid simulations rather than peptide identification benchmarks
  — SMITER offers separate fragmentation models for those molecule classes.
- Your goal is quantification benchmarking (co-elution/co-fragmentation effects) rather
  than identification-rate-vs-complexity curves.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SMITER
  role: Simulates the LC-MS/MS runs (peak properties, fragmentation, noise injection,
    mzML writing) for each peptide set size
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Language in which SMITER runs and in which the identification-rate analysis
    and plotting are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:e97e5756024d0b475ff49cc69a2ee8e3be7c7c3caa201ee384b0fa003f506eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# identification-rate-curve-analysis

## Summary

Reproduce the SMITER peptide-set-size benchmark by simulating yeast LC-MS/MS runs with increasing peptide counts, searching them with X!Tandem + Percolator, and plotting PSM identification rate (1% q-value) versus set size.

## When to use

Use when you need to characterize how simulated dataset complexity (number of peptides in a synthetic LC-MS/MS run) affects downstream peptide identification rates with a search-and-rescore pipeline.

## When NOT to use

- Your input is real experimental mzML data with unknown ground truth — SMITER is for creating synthetic gold-standard mzMLs from chemical formulas.
- You need nucleoside or lipid simulations rather than peptide identification benchmarks — SMITER offers separate fragmentation models for those molecule classes.
- Your goal is quantification benchmarking (co-elution/co-fragmentation effects) rather than identification-rate-vs-complexity curves.

## Inputs

- S. cerevisiae proteome (for in silico tryptic digestion and search database)
- csv file of peak properties from example_data (converted via smiter.lib.csv_to_peak_properties)
- DeepLC retention time predictions for peptides
- Simulated mzML files per peptide-set size

## Outputs

- One synthetic mzML per peptide set size (smiter.synthetic_mzml.write_mzml)
- PSMs rescored by Percolator at 1% q-value per set size
- identification_rate_curve.png (identification rate vs peptide set size)

## How to apply

Digest the S. cerevisiae proteome in silico with trypsin and build peak properties dicts for peptide sets of increasing size using smiter.lib.csv_to_peak_properties on the example_data csv. Predict retention times with DeepLC and configure simulation parameters: gradient length, peak width 30–120 s, dynamic exclusion 30 s, minimum intensity 500, and top20 DDA with at most 20 MS2 scans per MS1 cycle. For each set size, run the simulation with fragmentation_functions.PeptideFragmentor and smiter.noise_functions.UniformNoiseInjector, writing one mzML per set via smiter.synthetic_mzml.write_mzml. Search each simulated mzML against the yeast proteome with X!Tandem, rescore PSMs with Percolator, filter at 1% q-value, and compute the identification rate per set size. Plot identification rate versus peptide set size to reproduce the Figure 6 curve. Expect near-complete recovery for small sets: about 90% of 5,000 simulated peptides identified (many with multiple PSMs), dropping below 50% for sets larger than 130,000 peptides.

## Related tools

- **SMITER** (Simulates the LC-MS/MS runs (peak properties, fragmentation, noise injection, mzML writing) for each peptide set size) — https://github.com/LeidelLab/SMITER
- **Python** (Language in which SMITER runs and in which the identification-rate analysis and plotting are implemented)

## Examples

```
from smiter.lib import csv_to_peak_properties; from smiter.synthetic_mzml import write_mzml; peak_properties = csv_to_peak_properties('example_data.csv'); write_mzml(peak_properties, fragmentor, noise_injector, params)
```

## Evaluation signals

- About 90% of 5,000 simulated peptides are identified at 1% q-value, most through multiple PSMs
- Identification rate falls below 50% for peptide sets larger than ~130,000 peptides
- Each set size yields a valid mzML readable by X!Tandem and producing PSMs after Percolator rescoring
- The plotted curve matches the Figure 6 monotonic decay of identification rate with increasing set size
- Simulation parameters (peak width 30–120 s, dynamic exclusion 30 s, minimum intensity 500, top20 DDA, ≤20 MS2 scans per MS1 cycle) are applied consistently across all set sizes

## Limitations

- Identification rate degrades sharply as the simulated peptide set grows beyond ~130,000 peptides (below 50%), so results are size-dependent
- SMITER simulation quality depends on the noise and fragmentation models chosen (e.g. UniformNoiseInjector, PeptideFragmentor); alternative models may change the curve
- No changelog or discussion-section limitations were available in the article index, so failure modes beyond those documented are unknown

## Evidence

- [methods] task workflow — peak properties from csv: "create the peak properties dict (You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`)"
- [methods] task workflow — run simulation and write mzML: "Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [results] finding — high recovery at small set size: [evidence span withheld — anchor status: not_found]
- [readme] README — purpose of gold-standard datasets: [evidence span withheld — anchor status: external_doc]
- [readme] README — modular noise models: [evidence span withheld — anchor status: external_doc]
