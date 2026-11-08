---
name: sum-based-normalization-comparison-tic
description: Use when you have raw peak-intensity tables (e.g. xlsx) from pooled QC
  injections and must decide among iTIC, fTIC, and mTIC sum-based scaling before adopting
  ML-based normalizers like SERDA/SERRF.
when_to_use_negative:
- 'When the goal is full-panel normalization across all 563 metabolite annotations:
  the study showed no sum-based normalization achieved median RSD below 30% there,
  so use SERDA/SERRF instead.'
- When an external validation set (e.g. BioIVT validation pools) requires ML-based
  normalization rather than a simple sum-scaling comparison.
- When samples to be normalized (e.g. blanks) should be excluded — leave their cells
  empty rather than forcing them into the sum-based scaling.
tools:
- name: R 3.6.3
  role: Runtime for computing iTIC/fTIC/mTIC-normalized intensities and per-group
    median RSDs via the SERDA runner.R script
  repo: https://github.com/slfan2013/SERDA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:3887d33205d7fd6538fbdb39c3bb710e8f8066ea8caf17c631c31c579d31628d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - R 3.6.3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sum-based-normalization-comparison-tic

## Summary

Benchmark the three sum-based normalization schemes (iTIC, fTIC, mTIC) on QC-injection peak-intensity tables to pick the scheme with the lowest median relative standard deviation (RSD). In this GC-MS metabolomics study, mTIC won on the 16 amino acids, yielding 13.5% RSD for MSTFA and 8.3% RSD for MTBSTFA derivatization.

## When to use

Use when you have raw peak-intensity tables (e.g. xlsx) from pooled QC injections and must decide among iTIC, fTIC, and mTIC sum-based scaling before adopting ML-based normalizers like SERDA/SERRF.

## When NOT to use

- When the goal is full-panel normalization across all 563 metabolite annotations: the study showed no sum-based normalization achieved median RSD below 30% there, so use SERDA/SERRF instead.
- When an external validation set (e.g. BioIVT validation pools) requires ML-based normalization rather than a simple sum-scaling comparison.
- When samples to be normalized (e.g. blanks) should be excluded — leave their cells empty rather than forcing them into the sum-based scaling.

## Inputs

- xlsx peak-intensity table for BioIVT pooled plasma QC samples (GC-MS, derivatized with MSTFA and MTBSTFA)
- Feature annotation restricting the table to the 16 amino-acid features
- Per-injection ion-current sums used as iTIC/fTIC/mTIC scaling factors

## Outputs

- Median RSD (SD/mean x 100) per normalization method (iTIC, fTIC, mTIC) and per derivatization reagent
- Summary table replicating Supplementary Table S1 / Figure 3 identifying the best sum-based normalizer

## How to apply

Load the peak-intensity xlsx into R via the data_file path setting in the SERDA runner.R script (data_file = "filelocation.xlsx"), then restrict to the 16 amino-acid features. Compute the three normalization schemes from the summed ion-current scaling factors: iTIC (total ion current including all peaks), fTIC, and mTIC, dividing each feature by the corresponding per-injection sum. For each normalization and each derivatization reagent (MSTFA and MTBSTFA), compute median RSD (SD/mean x 100) across the BioIVT pooled QC injections. Assemble a summary table replicating Supplementary Table S1 / Figure 3 and confirm mTIC gives the lowest median RSD. Rationale: in the source study, neither internal standards nor sum-based normalizations reached <30% median RSD across all 563 metabolite annotations, motivating ML normalization, but for the small derivatization subset mTIC is the best sum-based choice (13.5% MSTFA, 8.3% MTBSTFA).

## Related tools

- **R 3.6.3** (Runtime for computing iTIC/fTIC/mTIC-normalized intensities and per-group median RSDs via the SERDA runner.R script) — https://github.com/slfan2013/SERDA

## Examples

```
Rscript SERDA_runner.R  # after setting data_file <- "filelocation.xlsx" in the script
```

## Evaluation signals

- Reproduced median mTIC RSD must match the reported values within rounding tolerance: 13.5% for MSTFA and 8.3% for MTBSTFA on the 16 amino acids
- mTIC must show the lowest median RSD among iTIC, fTIC, and mTIC for both derivatization reagents
- RSD must be computed as SD/mean x 100 across pooled QC injections, per normalization and per derivatization reagent
- Summary table should be reconcilable with Supplementary Table S1 / Figure 3
- Sanity check: across all 563 metabolite annotations, no sum-based normalization should drop below ~30% median RSD

## Limitations

- Sum-based normalizations (iTIC, fTIC, mTIC) could not achieve median precision below 30% across all 563 metabolite annotations in the full-scale study; their advantage is limited to subsets like the 16 amino acids
- Results are specific to derivatization-based GC-MS metabolomics (MSTFA and MTBSTFA) and the small BioIVT plasma pooled-QC dataset
- The runner script targets R 3.6.3 / Windows x64 with reticulate 1.19, keras 2.7.0, pROC 1.17.0.1, caret 6.0.86 and Python 3.6.10 configured via reticulate

## Evidence

- [abstract] Sum-based normalizations underperform across full feature panel: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [readme] Runner script execution path for loading xlsx in R: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [abstract] External validation context using BioIVT plasma pools: "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
