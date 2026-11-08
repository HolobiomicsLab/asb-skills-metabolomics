---
name: gc-ms-qc-precision-assessment
description: Use when GC-MS peak-area tables from plasma samples plus QC injections
  must be precision-scored as RSD — e.g. benchmarking derivatization agents (PCF/MTBSTFA/MSTFA),
  internal-standard schemes, or how QC sampling frequency affects normalization error.
when_to_use_negative:
- Samples you do not want normalized (e.g. blank samples) — per the README these must
  be left as empty cells rather than passed through the normalization/QC training
- Single-injection or QC-free datasets where RSD across biological samples cannot
  be computed
- Data from non-GC-MS, non-derivatization-based platforms where PCF/MTBSTFA/MSTFA
  derivatization-precision comparisons do not apply
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Runs the SERDA runner script and computes per-trial, per-metabolite RSDs of
    ISTD-normalized peak areas for each derivatization reagent
  repo: github.com/slfan2013/SERDA
- name: reticulate
  role: Bridges R 3.6.3 to Python 3.6.10 so the SERDA denoising autoencoder can be
    trained per QC-frequency configuration
  repo: github.com/slfan2013/SERDA
- name: keras
  role: Implements and trains the SERDA denoising autoencoder models used for each
    QC-placement-frequency configuration
  repo: github.com/slfan2013/SERDA
- name: gas chromatography-mass spectrometry (GC-MS)
  role: Derivatization-based acquisition platform producing the peak-area tables whose
    QC precision is assessed
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:372a2e752317a38e2d9527cdbc79fc1fe6155f2308f8fa1eb5f0e4802d4340d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - gas chromatography-mass spectrometry (GC-MS)
  - keras
  - reticulate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gc-ms-qc-precision-assessment

## Summary

Quantifies technical precision of GC-MS metabolomics normalization by computing relative standard deviations (RSD) across pooled-QC or BioIVT plasma samples after internal-standard or model-based normalization, then benchmarks results against reference values for derivatization agents (PCF, MTBSTFA, MSTFA) and QC-sampling densities.

## When to use

Use when GC-MS peak-area tables from plasma samples plus QC injections must be precision-scored as RSD — e.g. benchmarking derivatization agents (PCF/MTBSTFA/MSTFA), internal-standard schemes, or how QC sampling frequency affects normalization error.

## When NOT to use

- Samples you do not want normalized (e.g. blank samples) — per the README these must be left as empty cells rather than passed through the normalization/QC training
- Single-injection or QC-free datasets where RSD across biological samples cannot be computed
- Data from non-GC-MS, non-derivatization-based platforms where PCF/MTBSTFA/MSTFA derivatization-precision comparisons do not apply

## Inputs

- Raw GC-MS peak-area table (e.g. Supplementary Table S1) with metabolite and paired stable-isotope internal-standard columns for 30 BioIVT plasma samples in three monthly replicate trials under PCF, MTBSTFA, and MSTFA derivatization
- Pool QC rows and biological sample rows in xlsx format (e.g. 'SERRF example dataset - with validate.xlsx') with _batch_, _sampleType_, _time_, _label_ columns and _No_ compound index
- Independent external validation QC samples (BioIVT pooled plasma validation QC samples) for out-of-sample error estimation

## Outputs

- Per-metabolite RSD (%) values averaged across replicate trials per derivatization reagent, written to istd_aminoacid_rsd.csv
- Median RSD across metabolite annotations per QC-placement frequency (10/20/40/80), written to figure6_qc_frequency_median_rsd.csv
- Trained SERDA autoencoder models per QC-density configuration

## How to apply

Load the raw peak-area table (e.g. Supplementary Table S1: 16 amino acids with paired stable-isotope-labeled internal standards across 30 BioIVT plasma samples acquired in three monthly replicate trials per derivatization reagent). Apply one-to-one ISTD normalization in R by dividing each amino acid's raw peak area by its cognate internal-standard peak area for every injection, then compute per-metabolite RSD (%) across samples within each monthly trial and average the per-trial RSDs into one summary value per reagent. For QC-frequency sensitivity, subsample or duplicate pool-QC rows so one pool QC follows every 10, 20, 40, or 80 biological samples, train the SERDA denoising autoencoder per configuration (R 3.6.3 bridged via reticulate 1.19 to Python 3.6.10 with keras 2.7.0), and apply each model to independent BioIVT validation QC pools. Compare recomputed averages against reported references (PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%; QC-frequency median RSD 17%, 20%, 22%, 25% for 10/20/40/80) and write summary tables such as istd_aminoacid_rsd.csv and figure6_qc_frequency_median_rsd.csv. Judge success by reproducing the reference RSD values within reasonable tolerance and by the expected monotonic degradation of precision as QC density decreases.

## Related tools

- **R** (Runs the SERDA runner script and computes per-trial, per-metabolite RSDs of ISTD-normalized peak areas for each derivatization reagent) — github.com/slfan2013/SERDA
- **reticulate** (Bridges R 3.6.3 to Python 3.6.10 so the SERDA denoising autoencoder can be trained per QC-frequency configuration) — github.com/slfan2013/SERDA
- **keras** (Implements and trains the SERDA denoising autoencoder models used for each QC-placement-frequency configuration) — github.com/slfan2013/SERDA
- **gas chromatography-mass spectrometry (GC-MS)** (Derivatization-based acquisition platform producing the peak-area tables whose QC precision is assessed)

## Examples

```
R -e 'data_file <- "SERRF example dataset - with validate.xlsx"; source("SERDA runner.R")'
```

## Evaluation signals

- Recomputed average ISTD-normalized RSD per derivatization reagent matches the reported values: PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%
- Median RSD per QC-placement frequency matches the reported monotonic series 17%, 20%, 22%, 25% RSD for one QC after every 10, 20, 40, and 80 samples
- SERDA-normalized training QC samples reach ~16% median RSD and independent BioIVT validation QC pools ~19% RSD (vs 34% for SERRF), all below the >30% median precision of ISTD and sum-based normalization across 563 metabolite annotations
- Output CSVs (istd_aminoacid_rsd.csv, figure6_qc_frequency_median_rsd.csv) contain one summary row/value per reagent or QC frequency with no missing metabolite entries
- Environment check: R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10 (numpy 1.18.1) as pinned in the README reproduce the model behavior

## Limitations

- Neither internal standards nor sum-based normalizations achieved median precision below 30% across all 563 metabolite annotations, so ISTD-based RSD assessment alone cannot certify large-scale normalization quality
- Normalization precision degrades monotonically as QC sampling density decreases; below one pool QC per 10 biological samples, median error on independent BioIVT validation QCs rises toward 25% RSD
- The SERDA runner is pinned to a specific environment (R 3.6.3, reticulate 1.19, keras 2.7.0, Python 3.6.10, Windows 10 x64) and expects the xlsx input format with _batch_, _sampleType_, _time_, _label_, and _No_ columns; non-normalized samples (blanks) must be left as empty cells

## Evidence

- [abstract] neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [abstract] SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples: "SERDA lowered the median standard deviations of the training QC samples down to 16% RSD, yielding an overall error of 19% RSD when applied to the independent BioIVT validation QC samples"
- [abstract] external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD): "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [other] After normalization to each individual stable-isotope-labeled amino acid, PCF derivatization gave the lowest precision with 2.7% average RSD, followed by MTBSTFA at 8.9% RSD and MSTFA at 9.6% RSD.: [evidence span withheld — anchor status: not_found]
- [readme] Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines.: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
