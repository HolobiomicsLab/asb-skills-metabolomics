---
name: internal-standard-normalization-benchmarking
description: Use when you have GC-MS peak-intensity tables with pool QC samples (and
  ideally an independent external validation QC set) and need to quantify which normalization
  — sum-based iTIC/fTIC/mTIC, SERRF, or SERDA — minimizes technical variation.
when_to_use_negative:
- The input is not raw peak-intensity data with designated QC samples and batch/time
  metadata (e.g. it is already a feature table or has no QC structure), since RSD-based
  benchmarking cannot be computed
- The study is small-scale with few QC samples; the benchmarked performance (16–19%
  RSD) was established on >800 QC samples over 1.2 years of acquisition with seven
  column changes and may not transfer
- You only need one normalization applied, not a head-to-head comparison; use the
  SERRF or SERDA runner directly instead of the benchmarking workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: R 3.6.3
  role: Host environment in which SERDA runner.R is executed to compute normalizations
    and median RSDs
  repo: https://github.com/slfan2013/SERDA
- name: keras 2.7.0
  role: Implements the SERDA denoising autoencoder normalization being benchmarked
  repo: https://github.com/slfan2013/SERDA
- name: reticulate 1.19
  role: Bridges R 3.6.3 to Python 3.6.10 so keras-backed SERDA can run from the R
    script
- name: SERRF (Shiny-SERRF)
  role: Random-forest-based normalization baseline run alongside SERDA and internal-standard
    methods for RSD comparison; also available online at slfan.shinyapps.io/ShinySERRF/
  repo: https://github.com/slfan2013/Shiny-SERRF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:dfd75226c1f7276f7ce195f3458becccfd4d8e8366299e0c064d53bb1101039d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - R 3.6.3
  - SERRF (Shiny-SERRF)
  - keras 2.7.0
  - reticulate 1.19
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# internal-standard-normalization-benchmarking

## Summary

Benchmarks classic internal-standard-based normalizations (iTIC, fTIC, mTIC) against machine-learning methods (SERRF, SERDA) by computing median RSD over QC samples, as done in a large-scale GC-MS metabolomics study of 413 pool QC training plus 413 BioIVT validation samples over 563 metabolite annotations.

## When to use

Use when you have GC-MS peak-intensity tables with pool QC samples (and ideally an independent external validation QC set) and need to quantify which normalization — sum-based iTIC/fTIC/mTIC, SERRF, or SERDA — minimizes technical variation.

## When NOT to use

- The input is not raw peak-intensity data with designated QC samples and batch/time metadata (e.g. it is already a feature table or has no QC structure), since RSD-based benchmarking cannot be computed
- The study is small-scale with few QC samples; the benchmarked performance (16–19% RSD) was established on >800 QC samples over 1.2 years of acquisition with seven column changes and may not transfer
- You only need one normalization applied, not a head-to-head comparison; use the SERRF or SERDA runner directly instead of the benchmarking workflow

## Inputs

- QC peak-intensity table (xlsx) with pool QC training samples and external validation QC samples (e.g. 413 + 413 BioIVT plasma pools, 563 metabolite annotations)
- Sample metadata columns required by SERRF/SERDA: batch, sampleType (qc/sample/validate), time, label, and compound index No
- Example dataset: 'SERRF example dataset - with validate.xlsx' from the Shiny-SERRF repository

## Outputs

- Median RSD per normalization method (iTIC, fTIC, mTIC, SERRF, SERDA, post-SERDA sum) on pool QC and external validation QC samples
- Single CSV summary table assembling all median RSD values
- Normalized peak-intensity table produced by the chosen method

## How to apply

Load the QC peak-intensity table (e.g. 413 pool QC training plus 413 BioIVT validation samples across 563 metabolite annotations) into R by setting `data_file = "filelocation.xlsx"` in SERDA runner.R and running all script lines. Compute the internal-standard-based normalizations iTIC, fTIC, and mTIC and their median RSDs across all metabolite annotations — in this study these fell in the 53–63% range, establishing the floor baseline. Run SERRF normalization and record median RSD on pool QC samples (~19%) and on the external BioIVT validation pools (~34%), which reveals overfitting to training QCs. Run the SERDA denoising autoencoder normalization (keras 2.7.0 via reticulate 1.19 with Python 3.6.10 under R 3.6.3) and record median RSD on pool QC (~16%) and BioIVT validation (~17%). Finally, apply a post-SERDA sum normalization to the corrected table and recompute median RSDs to verify no further improvement, then assemble all values into a single CSV summary table.

## Related tools

- **R 3.6.3** (Host environment in which SERDA runner.R is executed to compute normalizations and median RSDs) — https://github.com/slfan2013/SERDA
- **keras 2.7.0** (Implements the SERDA denoising autoencoder normalization being benchmarked) — https://github.com/slfan2013/SERDA
- **reticulate 1.19** (Bridges R 3.6.3 to Python 3.6.10 so keras-backed SERDA can run from the R script)
- **SERRF (Shiny-SERRF)** (Random-forest-based normalization baseline run alongside SERDA and internal-standard methods for RSD comparison; also available online at slfan.shinyapps.io/ShinySERRF/) — https://github.com/slfan2013/Shiny-SERRF

## Examples

```
Rscript -e 'data_file <- "filelocation.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- iTIC/fTIC/mTIC median RSDs fall in the 53–63% range across all 563 metabolite annotations (no sum-based method below 30% median precision)
- SERRF reproduces ~19% median RSD on pool QC samples but degrades to ~34% on independent BioIVT validation pools
- SERDA reproduces ~16% median RSD on training pool QC and ~17% on BioIVT validation QC samples
- Post-SERDA sum normalization shows no further improvement in median RSD over SERDA alone
- All computed median RSD values assemble into the single CSV summary table matching the reported benchmarks

## Limitations

- The benchmark values were derived from a specific large-scale GC-MS setup (>4000 human plasma samples, 1.2 years acquisition, seven column changes); transfer to other platforms or scales is not guaranteed
- SERRF's 19% training QC precision masks overfitting — its external BioIVT validation RSD is 34%, so training-QC RSD alone is not a sufficient performance metric
- Internal-standard and sum-based normalizations could not achieve median precision below 30% across all 563 annotations in this dataset
- Environment is pinned: R 3.6.3, reticulate 1.19, keras 2.7.0, pROC 1.17.0.1, caret 6.0.86, Python 3.6.10, numpy 1.18.1 on Windows 10 x64; deviations may affect reproducibility
- Blank samples and other samples not to be normalized must be left with empty sampleType cells or they will be normalized

## Evidence

- [abstract] iTIC/fTIC/mTIC baseline range: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [abstract] SERRF vs SERDA performance: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] SERDA performance: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [methods] runner invocation: "oad `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] SERRF input format requirements: [evidence span withheld — anchor status: external_doc]
