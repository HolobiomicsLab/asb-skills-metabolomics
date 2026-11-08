---
name: paired-nonparametric-method-comparison
description: Use when two or more metabolomics normalization methods must be compared
  on identical GC-MS feature-intensity data with paired per-metabolite precision (RSD)
  values, to decide which method yields the highest coverage of compounds passing
  a <30% RSD threshold.
when_to_use_negative:
- Data are unpaired or come from different metabolite sets under each method — the
  Wilcoxon signed-rank test requires matched per-metabolite RSDs from the identical
  QC samples.
- The goal is per-sample normalization rather than benchmarking normalization methods
  against each other; use the individual method workflow (e.g. run SERDA runner.R
  or Shiny-SERRF) instead of a paired-comparison harness.
- Input is not a GC-MS feature-intensity matrix with pooled QC and validation blocks
  (e.g. it is already a statistical summary or has no QC structure) — the RSD/coverage
  metrics depend on repeated QC measurements.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Primary environment for running the comparison workflow (SERDA runner.R, caret,
    pROC) and computing Wilcoxon signed-rank tests and <30% RSD coverage percentages.
  repo: https://github.com/slfan2013/SERDA
- name: caret
  role: R package (v6.0.86) used in the SERDA workflow environment supporting cross-validation
    folds and model evaluation during the benchmark.
  repo: https://github.com/slfan2013/SERDA
- name: pROC
  role: R package (v1.17.0.1) loaded in the SERDA workflow environment, used for ROC-based
    evaluation alongside the RSD comparison.
  repo: https://github.com/slfan2013/SERDA
- name: keras
  role: R package (v2.7.0) providing the denoising autoencoder backend for SERDA normalization,
    one of the methods being compared.
  repo: https://github.com/slfan2013/SERDA
- name: reticulate
  role: R package (v1.19) that bridges R and Python 3.6.10 so the keras autoencoder
    can be invoked from SERDA runner.R.
  repo: https://github.com/slfan2013/SERDA
- name: Shiny-SERRF
  role: Random-forest-based SERRF normalization method used as a comparison baseline;
    runnable via app.R in RStudio or the online Shiny app.
  repo: https://github.com/slfan2013/Shiny-SERRF
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:602576359b6acb63e8eda82f1a3a8c26bc1c01b3488d9fe34fd49eb2add2efdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - Shiny-SERRF
  - caret
  - keras
  - pROC
  - reticulate
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# Paired nonparametric method comparison

## Summary

A statistical benchmarking skill for comparing metabolite normalization methods (e.g. SERDA, SERRF, batchwise-LOESS) by computing per-metabolite RSDs under five-fold cross-validation on training pooled QCs and external validation QC pools, then testing per-metabolite differences with a Wilcoxon signed-rank test and reporting the fraction of compounds meeting the <30% RSD precision threshold.

## When to use

Use when two or more metabolomics normalization methods must be compared on identical GC-MS feature-intensity data with paired per-metabolite precision (RSD) values, to decide which method yields the highest coverage of compounds passing a <30% RSD threshold.

## When NOT to use

- Data are unpaired or come from different metabolite sets under each method — the Wilcoxon signed-rank test requires matched per-metabolite RSDs from the identical QC samples.
- The goal is per-sample normalization rather than benchmarking normalization methods against each other; use the individual method workflow (e.g. run SERDA runner.R or Shiny-SERRF) instead of a paired-comparison harness.
- Input is not a GC-MS feature-intensity matrix with pooled QC and validation blocks (e.g. it is already a statistical summary or has no QC structure) — the RSD/coverage metrics depend on repeated QC measurements.

## Inputs

- GC-MS feature-intensity xlsx file containing training pooled QC samples plus external validation QC pools (e.g. BioIVT plasma pools)
- Per-method normalized RSD values for the same set of metabolite annotations (paired observations)
- Reference / example dataset: SERRF example dataset - with validate.xlsx

## Outputs

- Per-method CV and validation RSD values per metabolite
- Wilcoxon signed-rank test p-value comparing paired per-metabolite RSDs between methods
- Percentage of metabolite annotations meeting the <30% RSD precision threshold under each method
- benchmark_results.csv containing CV/validation RSDs, Wilcoxon p-value, and coverage percentages

## How to apply

Load the GC-MS feature-intensity xlsx (training pooled QC block plus external BioIVT validation QC block) into R. Normalize the identical dataset under each candidate method — e.g. run SERDA via `SERDA runner.R` (which invokes a keras denoising autoencoder through reticulate/Python 3.6.10), and SERRF (random-forest-based) and batchwise-LOESS on the same splits. For each method, compute normalized RSDs under five-fold cross-validation on the training QCs and on the held-out BioIVT validation QC pools. Because the same compounds are measured under both methods, the RSD values are paired; use a Wilcoxon signed-rank test to assess whether one method's per-metabolite RSDs are systematically lower than the other's. Report the Wilcoxon p-value and, for each method, the percentage of all metabolite annotations reaching the <30% RSD precision threshold (the article's stated cutoff), then write these coverage percentages and the test statistic to a results table such as benchmark_results.csv.

## Related tools

- **R** (Primary environment for running the comparison workflow (SERDA runner.R, caret, pROC) and computing Wilcoxon signed-rank tests and <30% RSD coverage percentages.) — https://github.com/slfan2013/SERDA
- **caret** (R package (v6.0.86) used in the SERDA workflow environment supporting cross-validation folds and model evaluation during the benchmark.) — https://github.com/slfan2013/SERDA
- **pROC** (R package (v1.17.0.1) loaded in the SERDA workflow environment, used for ROC-based evaluation alongside the RSD comparison.) — https://github.com/slfan2013/SERDA
- **keras** (R package (v2.7.0) providing the denoising autoencoder backend for SERDA normalization, one of the methods being compared.) — https://github.com/slfan2013/SERDA
- **reticulate** (R package (v1.19) that bridges R and Python 3.6.10 so the keras autoencoder can be invoked from SERDA runner.R.) — https://github.com/slfan2013/SERDA
- **Shiny-SERRF** (Random-forest-based SERRF normalization method used as a comparison baseline; runnable via app.R in RStudio or the online Shiny app.) — https://github.com/slfan2013/Shiny-SERRF

## Examples

```
Rscript -e 'source("SERDA runner.R")'  # after editing data_file = "filelocation.xlsx"
```

## Evaluation signals

- Coverage percentages (<30% RSD) should fall in the ranges reported for GC-MS: SERDA 72–98% versus SERRF 50–81% of all metabolite annotations.
- Median RSD on training pooled QCs should be near the reported values (SERDA ~16%, SERRF ~19%) and on external BioIVT validation pools (SERDA ~19%, SERRF ~34%).
- The Wilcoxon signed-rank test must be run on paired per-metabolite RSDs (same compounds, same QC samples) — pairing structure should be verifiable in the output table.
- benchmark_results.csv should contain columns for per-method CV RSDs, validation RSDs, the Wilcoxon p-value, and <30% RSD coverage percentages.
- Method coverage values should be computed across all 563 metabolite annotations, matching the article's stated annotation count.

## Limitations

- Results are specific to GC-MS data with pooled QC and external validation QC blocks; transferability to LC-MS or other data types is not established here.
- External validation relies on independent BioIVT plasma pools; if such external pools are unavailable, only training-QC cross-validated RSDs can be computed, which may overstate performance (SERRF showed 19% training vs 34% external median RSD).
- Neither internal standards nor sum-based normalizations reached <30% median precision across all 563 annotations, so benchmark results should not be extrapolated to those simpler methods without re-running them on the same splits.
- The comparison depends on the keras/reticulate Python environment (Python 3.6.10, numpy 1.18.1) matching the article's configuration; version mismatches may affect SERDA reproducibility.

## Evidence

- [abstract] The <30% RSD precision threshold is the stated cutoff for benchmarking coverage.: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [abstract] Paired per-metabolite RSD comparison between SERDA and SERRF under external validation.: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] SERDA's own training vs external validation RSD values used as benchmark reference points.: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [intro] Concrete R environment specification needed to reproduce the comparison.: "R version 3.6.3 (2020-02-29)"
- [intro] Concrete Python/keras configuration required to run the SERDA normalization being benchmarked.: "version:        3.6.10 |Anaconda, Inc.| (default, May  7 2020, 19:46:08) [MSC v.1916 64 bit (AMD64)]"
- [methods] Entry point for running the SERDA normalization method on the xlsx input within the comparison workflow.: "oad `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [methods] Example dataset format referenced for the SERRF baseline and the comparison input.: "An example dataset can be downloaded [here](https://github.com/slfan2013/Shiny-SERRF/raw/master/SERRF%20example%20dataset%20-%20with%20validate.xlsx)."
- [readme] README-specified input columns (_batch_, _sampleType_, _time_, _label_, _No_) needed for both SERRF and the comparison harness.: [evidence span withheld — anchor status: external_doc]
