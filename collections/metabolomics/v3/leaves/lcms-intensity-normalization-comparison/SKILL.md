---
name: lcms-intensity-normalization-comparison
description: Use when an LC/MS ion-feature matrix (>150 samples) has QC injections
  that can be withheld, and the goal is to test whether data-driven synthetic-QC drift
  correction matches experimental-QC normalization.
when_to_use_negative:
- The dataset contains no QC injections (experimental or identifiable run-order QC)
  whose RSD can serve as the drift-correction benchmark
- Input is raw mzML spectra rather than a processed ion-feature matrix — peak detection/alignment
  via mzLearn must run first before this comparison is meaningful
- The task requires MS/MS identification or de-isotoping — mzLearn output is at the
  ion MS1 level only and this skill evaluates normalization, not compound annotation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: mzLearn
  role: Runs data-driven drift correction that internally generates synthetic QC samples
    and builds intensity and rt drift maps, replacing experimental QC normalization;
    its ion-feature matrices are the input to all four normalization treatments
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:f4ab45aadccdfa8cc4c4792aa07bab1bb287aee612c62fcb87958bd2e034eefe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - mzLearn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lcms-intensity-normalization-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks four LC/MS intensity normalization treatments — mzLearn's synthetic-QC drift correction, withheld experimental-QC correction, total-ion-current (TIC) normalization, and no normalization — by computing relative standard deviation (RSD/CV×100) of feature intensities across hidden QC injections on each normalization-evaluation dataset.

## When to use

Use when an LC/MS ion-feature matrix (>150 samples) has QC injections that can be withheld, and the goal is to test whether data-driven synthetic-QC drift correction matches experimental-QC normalization.

## When NOT to use

- The dataset contains no QC injections (experimental or identifiable run-order QC) whose RSD can serve as the drift-correction benchmark
- Input is raw mzML spectra rather than a processed ion-feature matrix — peak detection/alignment via mzLearn must run first before this comparison is meaningful
- The task requires MS/MS identification or de-isotoping — mzLearn output is at the ion MS1 level only and this skill evaluates normalization, not compound annotation

## Inputs

- mzLearn MS1 ion-feature matrices for six normalization-evaluation datasets, each with more than 150 samples, derived from mzML files
- sample-role annotations identifying experimental QC injections to be withheld (hidden) from normalization processing

## Outputs

- qc_rsd_results.csv tabulating per-dataset and aggregate RSD values for the four conditions: synthetic QC, experimental QC, TIC, and raw
- per-treatment RSD values (coefficient of variation × 100) computed across hidden QC injections

## How to apply

Load the mzLearn ion-feature matrices for the normalization-evaluation datasets (each with more than 150 samples), keeping the experimental QC injections hidden from all processing. Run mzLearn's data-driven drift correction, which generates synthetic QC samples and constructs intensity and rt drift maps through iterative data-driven learning, on each dataset. Apply the comparators on the same matrices: correction using the withheld experimental QC samples, TIC normalization, and raw (no normalization) intensities. For each treatment compute RSD (coefficient of variation × 100) of feature intensities across the hidden QC injections per dataset, then aggregate the per-dataset RSDs side by side into qc_rsd_results.csv. Correct application means synthetic QC RSD (~17%) lands within about one percentage point of experimental-QC RSD (~16%), while TIC (~23%) and raw (~25%) are clearly worse, matching Fig. 3c / Supplementary Table 4.

## Related tools

- **mzLearn** (Runs data-driven drift correction that internally generates synthetic QC samples and builds intensity and rt drift maps, replacing experimental QC normalization; its ion-feature matrices are the input to all four normalization treatments) — github.com/ReviveMed/mzEmbed

## Examples

```
import pandas as pd; rsd = (qc_df.std() / qc_df.mean() * 100); rsd.to_csv('qc_rsd_results.csv')
```

## Evaluation signals

- Mean RSD across the six datasets reproduces the reported benchmarks: ~17% (synthetic QC), ~16% (experimental QC), ~23% (TIC), ~25% (raw), as shown in Fig. 3c / Supplementary Table 4
- Synthetic-QC RSD falls within roughly one percentage point of experimental-QC RSD, demonstrating comparable drift correction without experimental QC samples
- TIC and raw treatments show clearly worse RSD (~23% and ~25%), confirming the gap that normalization must close
- qc_rsd_results.csv contains all four conditions aggregated side by side, with RSD computed only on hidden QC injections withheld from every processing step
- RSD is defined as coefficient of variation × 100 and computed per dataset before aggregation

## Limitations

- mzLearn output is at the ion MS1 level; it does not perform de-isotoping or MS/MS identification, so normalization comparison cannot extend to identified metabolites without downstream workflows
- Public metabolomics repositories may lack standardized QC samples to estimate instrument drift, which is exactly the gap synthetic QC normalization is designed to address
- Iterative learning is computationally heavy, with analysis taking up to 2.5 days on a single machine for large datasets

## Evidence

- [results] synthetic-vs-experimental-QC benchmark: [evidence span withheld — anchor status: not_found]
- [readme] zero-parameter design: [evidence span withheld — anchor status: external_doc]
