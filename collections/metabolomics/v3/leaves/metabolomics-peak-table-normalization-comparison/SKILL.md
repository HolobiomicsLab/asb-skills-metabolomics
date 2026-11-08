---
name: metabolomics-peak-table-normalization-comparison
description: Use when a raw QC-annotated GC-MS peak-intensity table (e.g., .xlsx with
  pool QC and external validation QC samples) must be normalized and the choice among
  sum-based, SERRF, and SERDA normalization needs empirical benchmarking.
when_to_use_negative:
- The input is LC-MS proteomics-style data or another platform where the SERRF/SERDA
  column requirements (batch, sampleType, time, label, No) and GC-MS derivatization-specific
  drift structure do not apply.
- The dataset is small-scale with few QC samples or no external validation pools,
  so the training-vs-BioIVT cross-validation RSD comparison cannot be evaluated.
- The goal is peak detection or annotation rather than normalization — the skill operates
  downstream of an already-annotated peak-intensity table.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: R 3.6.3
  role: host environment for running SERDA runner.R and computing median RSDs
  repo: github.com/slfan2013/SERDA
- name: keras 2.7.0
  role: drives the SERDA denoising autoencoder from R via reticulate
  repo: github.com/slfan2013/SERDA
- name: reticulate 1.19
  role: R-Python bridge configuring Python 3.6.10 for keras in SERDA runner.R
  repo: github.com/slfan2013/SERDA
- name: SERRF
  role: random-forest-based normalization baseline compared against SERDA; runnable
    via the Shiny-SERRF app or local R
  repo: github.com/slfan2013/Shiny-SERRF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:0070672b532b23374e167dd8e5bfe20851747c594b2e100e93f512fbc16bd2cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R 3.6.3
  - SERRF
  - keras 2.7.0
  - reticulate 1.19
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# metabolomics-peak-table-normalization-comparison

## Summary

Benchmarks normalization strategies for large-scale untargeted GC-MS metabolomics — classic sum-based methods (iTIC, fTIC, mTIC), SERRF, and the SERDA denoising autoencoder — by computing median RSD on pool QC training and external validation samples. It quantifies how well each method corrects batch/technical drift across a 1.2-year, 563-metabolite annotation peak table.

## When to use

Use when a raw QC-annotated GC-MS peak-intensity table (e.g., .xlsx with pool QC and external validation QC samples) must be normalized and the choice among sum-based, SERRF, and SERDA normalization needs empirical benchmarking.

## When NOT to use

- The input is LC-MS proteomics-style data or another platform where the SERRF/SERDA column requirements (batch, sampleType, time, label, No) and GC-MS derivatization-specific drift structure do not apply.
- The dataset is small-scale with few QC samples or no external validation pools, so the training-vs-BioIVT cross-validation RSD comparison cannot be evaluated.
- The goal is peak detection or annotation rather than normalization — the skill operates downstream of an already-annotated peak-intensity table.

## Inputs

- QC peak-intensity table in .xlsx format (pool QC training samples plus external validation QC samples, e.g., 413 + 413 BioIVT samples over 563 metabolite annotations)
- SERRF-format input columns: batch, sampleType (qc/sample/validate), time (processing order), label (sample labels, row #4), and No (compound index, column B); blank samples left as empty cells

## Outputs

- Normalized peak-intensity table per method (iTIC, fTIC, mTIC, SERRF, SERDA, post-SERDA sum)
- Median RSD per normalization method for training QC and external validation QC samples
- Single CSV summary table of all median RSD values

## How to apply

Load the QC peak-intensity table (e.g., 413 pool QC training plus 413 BioIVT validation QC samples across 563 metabolite annotations) by setting `data_file = "filelocation.xlsx"` in SERDA runner.R and running all script lines under R 3.6.3 with reticulate 1.19 bridging to keras 2.7.0 on Python 3.6.10. First compute the internal-standard-based normalizations iTIC, fTIC, and mTIC and their median RSDs across all metabolite annotations (expected ~53–63%), establishing the baseline that sum-based methods fail to reach 30% median precision. Then run SERRF and compute median RSD on pool QC (~19%) and external validation pools (~34%), followed by SERDA, which lowers training QC RSD to ~16% and BioIVT validation RSD to ~17%. Finally apply a post-SERDA sum normalization and recompute median RSDs to verify no further improvement, then assemble all median RSD values into a single CSV summary table.

## Related tools

- **R 3.6.3** (host environment for running SERDA runner.R and computing median RSDs) — github.com/slfan2013/SERDA
- **keras 2.7.0** (drives the SERDA denoising autoencoder from R via reticulate) — github.com/slfan2013/SERDA
- **reticulate 1.19** (R-Python bridge configuring Python 3.6.10 for keras in SERDA runner.R) — github.com/slfan2013/SERDA
- **SERRF** (random-forest-based normalization baseline compared against SERDA; runnable via the Shiny-SERRF app or local R) — github.com/slfan2013/Shiny-SERRF

## Examples

```
Rscript -e 'data_file = "filelocation.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- Median RSD on pool QC training samples for SERDA matches the reported benchmark of 16%.
- Median RSD on independent BioIVT validation QC samples for SERDA matches 17% (overall error 19% RSD).
- SERRF reproduces ~19% median RSD on pool QC but degrades to ~34% on external BioIVT validation pools.
- iTIC/fTIC/mTIC sum-based normalizations land in the 53–63% median RSD range, i.e., none reaches 30% precision across all 563 annotations.
- Post-SERDA sum normalization shows no further improvement over SERDA alone.

## Limitations

- Results are environment-sensitive: the article pins R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10 (Anaconda, 64-bit Windows) and numpy 1.18.1; other versions may not reproduce the reported RSDs.
- SERRF's advantage did not generalize to external validation: 19% pool QC precision degraded to 34% RSD on BioIVT plasma pools, so training-QC RSD alone is insufficient evidence of method quality.
- Sample types that should not be normalized (e.g., blank samples) must be left as empty cells in the SERRF/SERDA input format.

## Evidence

- [abstract] SERDA benchmark: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [abstract] SERRF external validation: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] Sum-based baseline: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [readme] Runner invocation: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] Pinned software environment: "> packageVersion("keras")
[1] ‘2.7.0’
> packageVersion("caret")
[1] ‘6.0.86’"
