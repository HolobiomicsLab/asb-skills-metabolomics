---
name: peptide-spectrum-search-fdr-filtering
description: Use when a synthetic (SMITER-generated) mzML must be quality-checked
  by searching it against the source proteome and reporting a 1% q-value PSM identification
  rate per peptide-set size.
when_to_use_negative:
- When the input spectra are real experimental acquisitions rather than SMITER-simulated
  mzML with known ground truth — the identification-rate benchmark requires knowing
  which peptides were simulated.
- When the goal is simulating or writing synthetic mzML itself rather than downstream
  search/validation of existing simulated files.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SMITER
  role: Generates the synthetic mzML files (peak properties from csv, PeptideFragmentor,
    UniformNoiseInjector, write_mzml) that are then searched to measure identification
    rate
  repo: https://github.com/LeidelLab/SMITER
- name: X!Tandem
  role: Searches each simulated mzML against the yeast proteome to produce candidate
    PSMs
- name: Percolator
  role: Rescores X!Tandem PSMs so they can be filtered at 1% q-value
- name: Python
  role: Glue language for running the SMITER simulation, search orchestration, q-value
    filtering, and curve plotting
  repo: https://pypi.python.org/pypi/smiter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:854b4e186fc3302da0f18300fd3f4e80127de6bf0468949f12258fde33679f09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Percolator
  - Python
  - SMITER
  - X!Tandem
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# peptide-spectrum-search-fdr-filtering

## Summary

Validate SMITER-simulated LC-MS/MS runs by searching the synthetic mzML files with X!Tandem, rescoring PSMs with Percolator, and computing peptide identification rates at 1% q-value to benchmark how recovery changes with peptide-set size.

## When to use

Use when a synthetic (SMITER-generated) mzML must be quality-checked by searching it against the source proteome and reporting a 1% q-value PSM identification rate per peptide-set size.

## When NOT to use

- When the input spectra are real experimental acquisitions rather than SMITER-simulated mzML with known ground truth — the identification-rate benchmark requires knowing which peptides were simulated.
- When the goal is simulating or writing synthetic mzML itself rather than downstream search/validation of existing simulated files.

## Inputs

- SMITER-simulated mzML files (one per peptide-set size)
- In silico trypsin-digested S. cerevisiae proteome (search database)
- Peak properties csv from example_data (converted via smiter.lib.csv_to_peak_properties)
- DeepLC-predicted peptide retention times

## Outputs

- Percolator-rescored PSM lists filtered at 1% q-value per mzML
- Peptide identification rate per peptide-set size
- identification_rate_curve.png (identification rate vs. peptide set size, reproducing Figure 6)

## How to apply

First generate the synthetic data: digest the S. cerevisiae proteome in silico with trypsin, build peak properties dicts with smiter.lib.csv_to_peak_properties from the example_data csv, and for each peptide-set size write one mzML via smiter.synthetic_mzml.write_mzml using fragmentation_functions.PeptideFragmentor and smiter.noise_functions.UniformNoiseInjector, with gradient-length, peak-width 30–120 s, dynamic-exclusion 30 s, minimum-intensity 500 and top20 DDA (max 20 MS2 per MS1 cycle) parameters. Then search each simulated mzML against the yeast proteome with X!Tandem and rescore the resulting PSMs with Percolator. Filter rescored PSMs at 1% q-value and compute, for each peptide-set size, the fraction of simulated peptides recovered. The rationale is that SMITER's ground truth lets the search pipeline's recovery be measured exactly; the benchmark expects ~90% identification for 5,000-peptide sets and <50% for >130,000-peptide sets.

## Related tools

- **SMITER** (Generates the synthetic mzML files (peak properties from csv, PeptideFragmentor, UniformNoiseInjector, write_mzml) that are then searched to measure identification rate) — https://github.com/LeidelLab/SMITER
- **X!Tandem** (Searches each simulated mzML against the yeast proteome to produce candidate PSMs)
- **Percolator** (Rescores X!Tandem PSMs so they can be filtered at 1% q-value)
- **Python** (Glue language for running the SMITER simulation, search orchestration, q-value filtering, and curve plotting) — https://pypi.python.org/pypi/smiter

## Examples

```
from smiter.synthetic_mzml import write_mzml; write_mzml(peak_properties, fragmentor, noise_injector, params)  # then search mzML with X!Tandem and rescore PSMs with Percolator at 1% q-value
```

## Evaluation signals

- At 1% q-value, ~90% of 5,000 simulated peptides are identified, most through multiple PSMs
- Identification rate falls monotonically with peptide-set size, dropping below 50% for sets larger than ~130,000 peptides
- The plotted curve (identification_rate_curve.png) reproduces the shape of Figure 6
- Every reported PSM passed the Percolator 1% q-value threshold
- Each peptide-set size yields exactly one mzML written via smiter.synthetic_mzml.write_mzml

## Limitations

- Identification rate degrades sharply as simulated peptide-set size grows — small sets recover ~90% while >130,000-peptide sets recover <50% — so the benchmark result is strongly size-dependent
- The article's discussion section content is absent, so no published limitations or caveats text is available beyond the abstract and methods
- No changelog is available for the SMITER repository, so version-specific behavior of the simulation modules cannot be traced

## Evidence

- [methods] SMITER-simulated mzML searched with X!Tandem and Percolator at 1% q-value: "5. Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [abstract] SMITER simulates LC-MS/MS runs: "SMITER (Synthetic mzML writer) is a Python-based command-line tool designed to simulate liquid-chromatography-coupled tandem mass spectrometry LC-MS/MS runs."
- [readme] Ground truth from chemical-formula-based synthetic mzML enables gold-standard validation: "Python library to create synthetic mzMLs file based on chemical formulas. All molecules can be simulated du to abstraction to chemical formulas."
