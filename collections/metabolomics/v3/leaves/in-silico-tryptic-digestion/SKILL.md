---
name: in-silico-tryptic-digestion
description: 'Use when a simulated gold-standard LC-MS/MS dataset is needed: a known
  FASTA proteome must be converted into a finite, size-controlled peptide list so
  that identification recovery (PSMs at 1% q-value) can be measured against ground
  truth.'
when_to_use_negative:
- The input is already a peptide list or peak properties csv — go directly to csv_to_peak_properties
  rather than re-digesting
- The goal is identification of real, uncharacterized experimental mzML data — in-silico
  digestion is for generating gold-standard simulations with known ground truth, not
  for analyzing measured runs
- A non-tryptic or modified-proteome digestion is required beyond what the peptide
  fragmentation/RT setup (PeptideFragmentor, DeepLC) supports
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SMITER
  role: Python library that consumes the digested peptide set as peak properties,
    simulates the LC-MS/MS run (fragmentation, noise, retention-time-scaled elution),
    and writes the synthetic mzML
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Execution environment for the digestion, csv-to-peak-properties conversion,
    DeepLC retention-time prediction, and SMITER simulation scripts
  repo: https://pypi.python.org/pypi/smiter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:5c67ea23d150ac9e5d7f99d0af93fbb20807732e95aac34f4cc3023b76eddc45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# In-silico tryptic digestion

## Summary

Digest a reference proteome (e.g. S. cerevisiae) in silico with trypsin to generate defined peptide sets of controlled size, which serve as the ground-truth input for building SMITER peak properties dicts and benchmarking identification rates in simulated LC-MS/MS runs.

## When to use

Use when a simulated gold-standard LC-MS/MS dataset is needed: a known FASTA proteome must be converted into a finite, size-controlled peptide list so that identification recovery (PSMs at 1% q-value) can be measured against ground truth.

## When NOT to use

- The input is already a peptide list or peak properties csv — go directly to csv_to_peak_properties rather than re-digesting
- The goal is identification of real, uncharacterized experimental mzML data — in-silico digestion is for generating gold-standard simulations with known ground truth, not for analyzing measured runs
- A non-tryptic or modified-proteome digestion is required beyond what the peptide fragmentation/RT setup (PeptideFragmentor, DeepLC) supports

## Inputs

- Reference proteome (e.g. S. cerevisiae) in FASTA or csv form
- csv of peptide formulas / peak properties (as in SMITER example_data)
- Chosen peptide set size (e.g. 5,000 peptides, or larger sets up to >130,000)

## Outputs

- In-silico tryptic peptide list per set size (ground-truth peptide IDs)
- Peak properties dict built via smiter.lib.csv_to_peak_properties
- One synthetic mzML per peptide set size (via smiter.synthetic_mzml.write_mzml)
- Identification-rate values per set size for the Figure 6 curve (identification_rate_curve.png)

## How to apply

Digest the target proteome in silico using trypsin specificity to enumerate candidate peptides, then select subsets of increasing size (e.g. 5,000 peptides up to >130,000) to control simulation complexity. Convert the peptide list (csv, as in SMITER's example_data) into a peak properties dict with smiter.lib.csv_to_peak_properties, and predict retention times with DeepLC before configuring general simulation params such as gradient_length, peak width 30–120 s, dynamic exclusion 30 s, minimum intensity 500, and top20 DDA with at most 20 MS2 scans per MS1 cycle. Simulate each peptide set with fragmentation_functions.PeptideFragmentor and smiter.noise_functions.UniformNoiseInjector, writing one mzML per set via smiter.synthetic_mzml.write_mzml. Because every simulated peptide is known by construction, the resulting mzML plus the original peptide list form a gold-standard pair for downstream search benchmarking (X!Tandem + Percolator, 1% q-value filtering).

## Related tools

- **SMITER** (Python library that consumes the digested peptide set as peak properties, simulates the LC-MS/MS run (fragmentation, noise, retention-time-scaled elution), and writes the synthetic mzML) — https://github.com/LeidelLab/SMITER
- **Python** (Execution environment for the digestion, csv-to-peak-properties conversion, DeepLC retention-time prediction, and SMITER simulation scripts) — https://pypi.python.org/pypi/smiter

## Examples

```
from smiter.lib import csv_to_peak_properties; peak_properties = csv_to_peak_properties('yeast_tryptic_peptides.csv')
```

## Evaluation signals

- Peak properties dict contains the expected number of peptide entries for each chosen set size (e.g. ~5,000 for the small benchmark set)
- Each written mzML file is parseable and corresponds to exactly one peptide set size with known ground-truth peptide identities
- Search of the smallest simulated set (5,000 peptides) against the yeast proteome recovers ~90% of simulated peptides at 1% q-value, most via multiple PSMs
- Identification rate decreases as peptide set size grows toward >130,000 peptides (reproducing the Figure 6 curve, saved as identification_rate_curve.png)

## Limitations

- Identification rates depend on simulation parameters (gradient_length, peak width 30–120 s, dynamic exclusion 30 s, minimum intensity 500, top20 DDA with ≤20 MS2 scans per MS1 cycle) — changing them shifts the recovery curve
- SMITER's simulation fidelity is bounded by its default noise and fragmentation models (e.g. UniformNoiseInjector, PeptideFragmentor), which may not capture all real-instrument effects
- No changelog or discussion-section limitations were available in the indexed article content, so failure modes beyond the documented workflow are unassessed

## Evidence

- [methods] csv_to_peak_properties: "You can convert a csv file (as in example_data) to csv using `smiter.lib.csv_to_peak_properties`"
- [methods] fragmentor choice: "Choose a fragmentor (e.g. fragmentation_functions.PeptideFragmentor)"
- [methods] mzML writing: "write the resulting mzML using `smiter.synthetic_mzml.write_mzml`"
- [readme] SMITER purpose: "SMITER (Synthetic mzML writer) is a python-based command-line tool designed to simulate LC-MS/MS runs."
