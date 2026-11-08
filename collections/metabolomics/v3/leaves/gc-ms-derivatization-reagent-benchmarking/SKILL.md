---
name: gc-ms-derivatization-reagent-benchmarking
description: Decision support for choosing between iTIC, fTIC, and mTIC sum-based
  normalizations in GC-MS metabolomics when derivatization reagent (MSTFA vs MTBSTFA)
  effects on precision need to be quantified.
when_to_use_negative:
- The goal is large-scale normalization across all 563 metabolite annotations rather
  than benchmarking sum-based methods on a small feature subset — the article reports
  neither internal standards nor sum-based normalizations achieved median precision
  below 30% at that scale, so SERDA/SERRF is the appropriate skill there
- Input is already normalized data without raw summed ion currents, so iTIC/fTIC/mTIC
  scaling factors cannot be recomputed
- The dataset lacks pooled QC injections injected repeatedly under both derivatization
  reagents, making RSD-based precision comparison undefined
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: R environment (v3.6.3 in the paper) used to load the xlsx peak-intensity table
    via SERDA runner.R, compute the three sum-based normalizations, and calculate
    median RSD per derivatization reagent
  repo: https://github.com/slfan2013/SERDA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:c434e974cecec329a6425bcff1b02be3e8133c331026c76cf4452e3d19d22ce2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gc-ms-derivatization-reagent-benchmarking

## Summary

Benchmarks the three sum-based normalization methods (iTIC, fTIC, and mTIC) for derivatization-based GC-MS metabolomics on amino-acid features, comparing median RSD across pooled QC samples per derivatization reagent (MSTFA vs MTBSTFA) to select the best baseline normalization.

## When to use

Decision support for choosing between iTIC, fTIC, and mTIC sum-based normalizations in GC-MS metabolomics when derivatization reagent (MSTFA vs MTBSTFA) effects on precision need to be quantified.

## When NOT to use

- The goal is large-scale normalization across all 563 metabolite annotations rather than benchmarking sum-based methods on a small feature subset — the article reports neither internal standards nor sum-based normalizations achieved median precision below 30% at that scale, so SERDA/SERRF is the appropriate skill there
- Input is already normalized data without raw summed ion currents, so iTIC/fTIC/mTIC scaling factors cannot be recomputed
- The dataset lacks pooled QC injections injected repeatedly under both derivatization reagents, making RSD-based precision comparison undefined

## Inputs

- Peak-intensity xlsx table for BioIVT pooled plasma validation QC samples (e.g. loaded via data_file path in SERDA runner.R)
- Feature annotation identifying the 16 amino-acid features
- Summed ion-current scaling factors for iTIC, fTIC, and mTIC normalization
- Sample metadata tagging QC injections by derivatization reagent (MSTFA vs MTBSTFA)

## Outputs

- Median RSD (SD/mean x 100) per normalization method (iTIC, fTIC, mTIC) per derivatization reagent
- Summary table replicating Supplementary Table S1 / Figure 3
- Selected best-performing normalization (mTIC: 13.5% RSD MSTFA, 8.3% RSD MTBSTFA)

## How to apply

Load the peak-intensity table (xlsx) for pooled QC samples into R by setting `data_file = "filelocation.xlsx"` in SERDA runner.R, then restrict to the 16 amino-acid features. Compute mTIC-, iTIC-, and fTIC-normalized intensities using the summed ion-current scaling factors for each normalization scheme. For each normalization and each derivatization reagent (MSTFA and MTBSTFA), compute the median RSD (SD/mean x 100) across QC injections, then assemble a summary table replicating Supplementary Table S1 / Figure 3. Decision point: the reagent/normalization combination with the lowest median RSD is preferred — in the BioIVT plasma dataset, mTIC yielded 13.5% RSD for MSTFA and 8.3% RSD for MTBSTFA, outperforming iTIC and fTIC for correcting errors across the 16 amino acids.

## Related tools

- **R** (R environment (v3.6.3 in the paper) used to load the xlsx peak-intensity table via SERDA runner.R, compute the three sum-based normalizations, and calculate median RSD per derivatization reagent) — https://github.com/slfan2013/SERDA

## Examples

```
Rscript SERDA_runner.R  # after editing data_file = "filelocation.xlsx" to point at the BioIVT pooled QC peak-intensity table
```

## Evaluation signals

- Reproduced median mTIC RSD values must match the reported 13.5% (MSTFA) and 8.3% (MTBSTFA) within rounding tolerance
- mTIC must yield the lowest median RSD of the three sum-based normalizations (iTIC, fTIC, mTIC) for both MSTFA and MTBSTFA
- Summary table matches Supplementary Table S1 / Figure 3 structure (normalization x derivatization reagent grid of median RSDs)
- RSD computation is SD/mean x 100 computed across QC injections per amino-acid feature, then median taken across the 16 amino acids

## Limitations

- Benchmark is based on a small BioIVT plasma derivatization set restricted to 16 amino acids — the finding does not extend to all 563 metabolite annotations, where sum-based normalizations yielded median precision above 30%
- Precision here is pooled-QC RSD; external cross-validation with BioIVT plasma pools yielded a median 34% RSD even for learned normalizations, so sum-based RSDs underestimate end-to-end batch-correction error
- Results are specific to MSTFA and MTBSTFA derivatization chemistries on GC-MS plasma metabolomics; other reagents or matrices are not covered by the reported numbers

## Evidence

- [results] mTIC outperforms iTIC and fTIC: [evidence span withheld — anchor status: not_found]
- [readme] Workflow mechanics: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [abstract] Scale limitation of sum-based normalization: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [abstract] External validation RSD: "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
