---
name: qc-median-rsd-precision-analysis
description: Use when comparing normalization strategies (ISTD, iTIC/fTIC/mTIC, SERRF,
  SERDA) on pooled QC samples and you need a per-method precision number as median
  RSD over compounds.
when_to_use_negative:
- The goal is absolute quantification or calibration rather than precision assessment
  across QC injections
- The dataset lacks designated pooled QC samples or sampleType labels, so per-compound
  RSD across QCs cannot be computed
- You intend to normalize blanks or validate samples — the input format specifies
  these should be left as empty cells and not normalized
tools:
- name: R 3.6.3
  role: Environment in which RSDs are computed per compound across QC injections and
    median RSDs are compared per normalization method
- name: SERDA runner.R
  role: Script that loads the data_file xlsx and produces SERDA-normalized peak areas
    whose QC RSDs are then evaluated; also reused to load tables for the mTIC/iTIC/fTIC
    comparisons
  repo: https://github.com/slfan2013/SERDA
- name: keras 2.7.0
  role: Python deep-learning backend (via reticulate) that runs the SERDA denoising
    autoencoder normalization being scored
- name: reticulate 1.19
  role: Bridges R and the Python 3.6.10/keras backend required to execute SERDA normalization
    before RSD assessment
  repo: https://rstudio.github.io/reticulate/index.html
- name: Shiny-SERRF
  role: Source of the SERRF-normalized peak areas and the example dataset used as
    the RSD comparison baseline
  repo: https://github.com/slfan2013/Shiny-SERRF
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:801da4a0fe0117299e3379c109d8901b1dd62890ab2a5df98d9dbe6d8b41a97e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - R 3.6.3
  - SERDA runner.R
  - Shiny-SERRF
  - keras 2.7.0
  - reticulate 1.19
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# qc-median-rsd-precision-analysis

## Summary

Computes per-compound relative standard deviation (RSD = SD/mean × 100) across pooled QC injections for GC-MS metabolite features, then takes the median RSD per normalization method as the precision metric. This is how the SERDA paper benchmarked internal-standard, sum-based (iTIC/fTIC/mTIC), SERRF, and SERDA normalization.

## When to use

Use when comparing normalization strategies (ISTD, iTIC/fTIC/mTIC, SERRF, SERDA) on pooled QC samples and you need a per-method precision number as median RSD over compounds.

## When NOT to use

- The goal is absolute quantification or calibration rather than precision assessment across QC injections
- The dataset lacks designated pooled QC samples or sampleType labels, so per-compound RSD across QCs cannot be computed
- You intend to normalize blanks or validate samples — the input format specifies these should be left as empty cells and not normalized

## Inputs

- Peak-intensity table (xlsx) with columns batch, sampleType (qc/sample/validate), time, label, and No compound index (SERRF/SERDA input format)
- Assignment of pooled QC injections and sample labels
- Compound lists defining the comparison subset (e.g., 563 annotations, 16 amino acids, 23 endogenous metabolites, 20 sugars/sugar alcohols)

## Outputs

- Per-compound RSD values across QC injections for each normalization method
- Median RSD per method per subset (e.g., rsd_comparison.csv)
- Comparison plot of median RSD by normalization method (e.g., figure7_rsd_comparison.png)

## How to apply

Load the peak-intensity table (xlsx) into R and split rows by sampleType (qc vs sample; blanks are left empty and not normalized). For each compound, compute RSD = SD/mean × 100 across the pooled QC injections, then take the median RSD over the compound set (e.g., all 563 annotations, a 16-amino-acid derivatization subset, or 23 endogenous metabolites paired one-to-one with their deuterated ISTDs). Repeat per normalization method and per sub-experiment (MSTFA vs MTBSTFA derivatization), then compare medians against reported benchmarks: mTIC 13.5% (MSTFA) and 8.3% (MTBSTFA) for amino acids; one-to-one ISTD 19%, SERDA 3.8%, SERRF 5% for the 23-compound set; SERRF 19% on training QCs but 34% on external BioIVT validation pools, SERDA 16%/19%. Rationale: median RSD across compounds is the paper's canonical technical-error metric because normalization quality varies strongly per metabolite, so medians over feature subsets reveal which compounds remain outliers (e.g., sorbitol-d8 one-to-class normalization restricted to the 20 sugars/sugar alcohols). Replicate comparison figures (Figure 3, Figure 7a, Supplementary Figure S2) and export summary tables such as rsd_comparison.csv.

## Related tools

- **R 3.6.3** (Environment in which RSDs are computed per compound across QC injections and median RSDs are compared per normalization method)
- **SERDA runner.R** (Script that loads the data_file xlsx and produces SERDA-normalized peak areas whose QC RSDs are then evaluated; also reused to load tables for the mTIC/iTIC/fTIC comparisons) — https://github.com/slfan2013/SERDA
- **keras 2.7.0** (Python deep-learning backend (via reticulate) that runs the SERDA denoising autoencoder normalization being scored)
- **reticulate 1.19** (Bridges R and the Python 3.6.10/keras backend required to execute SERDA normalization before RSD assessment) — https://rstudio.github.io/reticulate/index.html
- **Shiny-SERRF** (Source of the SERRF-normalized peak areas and the example dataset used as the RSD comparison baseline) — https://github.com/slfan2013/Shiny-SERRF

## Examples

```
Rscript SERDA_runner.R  # after editing data_file = "SERRF example dataset - with validate.xlsx"; then compute per-compound sd(x)/mean(x)*100 across QC rows
```

## Evaluation signals

- Median RSD for the 23 endogenous compounds must match reported values: one-to-one ISTD ≈ 19%, SERDA ≈ 3.8%, SERRF ≈ 5% (Figure 7a)
- mTIC median RSD for the 16 amino acids must match 13.5% (MSTFA) and 8.3% (MTBSTFA) within rounding tolerance (Supplementary Table S1 / Figure 3)
- SERRF on pooled QCs must reproduce ≈19% median RSD while external BioIVT validation pools give ≈34%; SERDA must give 16% training / 19% external validation
- Across all 563 metabolite annotations, neither ISTD nor sum-based normalization should reach a median precision below 30% — values below that flag a replication error
- Sorbitol-d8 one-to-class normalization restricted to the 20 sugars/sugar alcohols must underperform SERDA and SERRF, identifying its outlier status

## Limitations

- SERRF precision estimated on training pooled QCs (19%) is optimistic relative to external BioIVT validation pools (34%); QC-only RSD can overstate real-world precision
- One-to-one ISTD and one-to-class ISTD (e.g., sorbitol-d8 to sugars) can fail badly for specific compounds, so median RSD alone can hide outlier metabolites
- Reproduction requires a pinned environment (R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10/numpy 1.18.1); other versions may not regenerate the reported RSDs

## Evidence

- [abstract] SERRF vs SERDA training/external precision benchmark: "SERRF algorithm gave 19% median precision based on the pooled quality control samples, external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] SERDA training and external validation RSD targets: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [methods] Runner script invocation procedure: "oad `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [readme] sampleType labeling rules defining QC rows used for RSD computation: [evidence span withheld — anchor status: external_doc]
