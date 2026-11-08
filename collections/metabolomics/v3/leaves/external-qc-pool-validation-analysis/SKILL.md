---
name: external-qc-pool-validation-analysis
description: Use when a normalization was tuned on in-run pooled QCs and an independent
  commercial/reference plasma QC pool exists to confirm the method's error generalizes
  outside the training batches.
when_to_use_negative:
- No independent external QC material is available — five-fold CV on training pooled
  QCs alone cannot detect batch overfitting
- The external pools were acquired in the same batches as the training QCs, defeating
  the purpose of independent validation
tools:
- name: SERDA runner.R
  role: Applies the fitted denoising autoencoder normalization to the dataset containing
    the external BioIVT validation block, producing corrected intensities for RSD
    computation
  repo: https://github.com/slfan2013/SERDA
- name: SERRF
  role: Baseline random-forest normalizer whose external-QC RSD (34% vs SERDA's 17%)
    is compared against during validation
  repo: https://github.com/slfan2013/Shiny-SERRF
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:a4c628b6805709752f0fc470fd7da109e6e00a7ea40187cf00a791208ae9899e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - SERDA runner.R
  - SERRF
  - caret
  - pROC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# external-qc-pool-validation-analysis

## Summary

Validates a metabolomics normalization method by applying it to an independent external pooled-QC set (e.g. BioIVT plasma pools) and computing median RSDs, exposing overfitting that training-set cross-validation alone would hide.

## When to use

Use when a normalization was tuned on in-run pooled QCs and an independent commercial/reference plasma QC pool exists to confirm the method's error generalizes outside the training batches.

## When NOT to use

- No independent external QC material is available — five-fold CV on training pooled QCs alone cannot detect batch overfitting
- The external pools were acquired in the same batches as the training QCs, defeating the purpose of independent validation

## Inputs

- Normalized or raw peak-intensity table including an external validation QC block (e.g. 413 BioIVT pooled plasma QCs, 563 metabolite annotations)
- A normalization model already fitted on training pooled QCs (e.g. SERDA autoencoder weights from SERDA runner.R)

## Outputs

- Per-metabolite RSD values on the external validation QC samples
- Median external RSD (e.g. 17% SERDA vs 34% SERRF on BioIVT pools)
- Training-vs-external RSD comparison quantifying generalization gap

## How to apply

After tuning the normalizer (e.g. SERDA's denoising autoencoder or SERRF's random forest) on training pooled QCs under five-fold cross-validation, apply the frozen model to a held-out external QC pool set — in this study, 413 BioIVT pooled plasma validation QCs spanning the same 563 metabolite annotations. Compute per-metabolite RSDs on the corrected intensities of these external pools, then compare the median RSD against the training-QC median: SERRF degraded from 19% to 34% RSD on BioIVT pools, whereas SERDA held 16% → 17%, demonstrating genuine generalization. Report the external median RSD alongside the training value so the gap (or lack thereof) quantifies overfitting to acquisition batches.

## Related tools

- **SERDA runner.R** (Applies the fitted denoising autoencoder normalization to the dataset containing the external BioIVT validation block, producing corrected intensities for RSD computation) — https://github.com/slfan2013/SERDA
- **SERRF** (Baseline random-forest normalizer whose external-QC RSD (34% vs SERDA's 17%) is compared against during validation) — https://github.com/slfan2013/Shiny-SERRF

## Examples

```
Rscript SERDA_runner.R  # with data_file = "filelocation.xlsx" containing qc and validate sampleType rows
```

## Evaluation signals

- External (BioIVT) median RSD should stay close to the training pooled-QC median — SERDA: 16% → 17%; a jump like SERRF's 19% → 34% flags overfitting
- RSDs are computed over the same 563 metabolite annotations used in training so the comparison is like-for-like
- Independent external QCs (e.g. BioIVT plasma pools) are never used during model fitting or cross-validation
- Post-normalization sum scaling on SERDA output should show no further RSD improvement, confirming the external error is stable

## Limitations

- Requires acquisition of independent commercial/reference QC pools (BioIVT) beyond standard in-run pooled QCs, adding cost and planning
- External validation reflects the specific plasma matrix and GC-MS platform of the training study; generalization to other matrices or instruments is not demonstrated here

## Evidence

- [abstract] external QC validation distinguishes generalization from overfitting: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] SERDA retains low error on external validation pools: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [readme] run the normalization script on the full dataset including validation block: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] input file must flag external validation samples distinctly: [evidence span withheld — anchor status: external_doc]
