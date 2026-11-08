---
name: synthetic-qc-drift-correction
description: Use when raw LC/MS mzML data lacks standardized experimental QC samples
  (common for public metabolomics datasets) yet feature intensities must be corrected
  for batch, run-order, and intensity drift before statistical analysis or pre-training
  a model like the VAE.
when_to_use_negative:
- Input is already a normalized feature table (e.g. the 2,736-column CSV matrix accepted
  by mzEmbed) — re-running drift correction is unnecessary and would duplicate mzLearn's
  preprocessing.
- A study provides its own standardized experimental QC samples and a traditional
  QC-based normalization pipeline is preferred — synthetic QC is a replacement for,
  not an addition to, experimental QC correction.
- Downstream analysis requires de-isotoping or MS/MS identification — mzLearn results
  are at the ion MS1 level only and mapping ion features to MS/MS spectra must be
  carried out with downstream workflows.
- The LC mode is not HILIC positive mode (e.g. reverse-phase or negative mode) — the
  current pre-trained drift model was developed specifically for HILIC positive mode.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: mzLearn
  role: Core algorithm that runs from mzML files, generates synthetic QC samples,
    and constructs intensity/rt drift maps to normalize feature intensities without
    user-set parameters or QC samples.
  repo: github.com/ReviveMed/mzEmbed
- name: mzEmbed
  role: Codebase exposing mzLearn's outputs as input to downstream VAE pre-training/finetuning;
    the synthetic-QC-corrected 2,736-feature matrix is the expected input.
  repo: github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Comparator signal-detection method used as a benchmark to show mzLearn's drift
    correction yields higher signal detection (11,442 vs 7,100 on average) and comparable
    RSD.
- name: ASARI
  role: Comparator signal-detection method used as a benchmark (4,655 signals on average)
    when evaluating the drift-corrected feature output.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:a67712cf4a2a7860306b053e69649348035fb0d6f03bc4eaf3a0e7981db84e3d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - ASARI
  - XCMS
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# synthetic-qc-drift-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-driven drift-correction skill that normalizes LC/MS feature intensities across large cohorts using mzLearn's internally generated synthetic QC samples instead of experimental QC injections, achieving RSD comparable to QC-based normalization without requiring pooled or blank QC runs.

## When to use

Use when raw LC/MS mzML data lacks standardized experimental QC samples (common for public metabolomics datasets) yet feature intensities must be corrected for batch, run-order, and intensity drift before statistical analysis or pre-training a model like the VAE.

## When NOT to use

- Input is already a normalized feature table (e.g. the 2,736-column CSV matrix accepted by mzEmbed) — re-running drift correction is unnecessary and would duplicate mzLearn's preprocessing.
- A study provides its own standardized experimental QC samples and a traditional QC-based normalization pipeline is preferred — synthetic QC is a replacement for, not an addition to, experimental QC correction.
- Downstream analysis requires de-isotoping or MS/MS identification — mzLearn results are at the ion MS1 level only and mapping ion features to MS/MS spectra must be carried out with downstream workflows.
- The LC mode is not HILIC positive mode (e.g. reverse-phase or negative mode) — the current pre-trained drift model was developed specifically for HILIC positive mode.

## Inputs

- Raw LC/MS data files in the open-source mzML format
- Large-scale untargeted metabolomics cohorts (e.g. >150 samples per dataset; up to 2,075 files in a single run)
- Feature matrices lacking experimental QC injections (hidden QC samples withheld)

## Outputs

- Two-dimensional table of detected ion features defined by median rt and m/z values, with normalized intensities across samples
- Intensity and retention-time drift maps learned from the data
- qc_rsd_results.csv comparing RSD of synthetic-QC vs experimental-QC vs TIC vs raw normalization

## How to apply

Run mzLearn on raw mzML files: the algorithm iteratively learns signal characteristics, generates synthetic QC samples, and constructs intensity and retention-time drift maps through a dynamic, data-driven learning process. It autonomously refines signal detection and corrects rt and intensity drifts caused by batch effects and run order, without requiring input parameters or QC samples. The output is a two-dimensional table of detected ion features (median rt and m/z) with normalized intensities across samples. Rationale: because public repositories lack standardized QC samples to estimate instrument drift, synthetic QC replacement is the only viable normalization for large-cohort pre-training, e.g. the 2,736-feature matrix from 20,548 blood samples. Correctness is judged by comparing the RSD (coefficient of variation × 100) of feature intensities across hidden experimental QC injections against reported benchmarks (17% synthetic-QC vs 16% experimental-QC, 23% TIC, 25% raw).

## Related tools

- **mzLearn** (Core algorithm that runs from mzML files, generates synthetic QC samples, and constructs intensity/rt drift maps to normalize feature intensities without user-set parameters or QC samples.) — github.com/ReviveMed/mzEmbed
- **mzEmbed** (Codebase exposing mzLearn's outputs as input to downstream VAE pre-training/finetuning; the synthetic-QC-corrected 2,736-feature matrix is the expected input.) — github.com/ReviveMed/mzEmbed
- **XCMS** (Comparator signal-detection method used as a benchmark to show mzLearn's drift correction yields higher signal detection (11,442 vs 7,100 on average) and comparable RSD.)
- **ASARI** (Comparator signal-detection method used as a benchmark (4,655 signals on average) when evaluating the drift-corrected feature output.)

## Examples

```
git clone git@github.com:ReviveMed/mzEmbed.git && cd mzEmbed && pip install -r requirements.txt && python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --trials 40 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Per-dataset RSD (coefficient of variation × 100) across hidden QC injections should fall near the reported benchmark of 17% for synthetic-QC normalization vs 16% for experimental-QC normalization.
- The four-treatment comparison (synthetic QC, experimental QC, TIC, raw) should reproduce the reported means of 17%, 16%, 23%, and 25% respectively (Fig. 3c / Supplementary Table 4).
- Feature table shape must be [n_samples, n_features] with columns carrying median rt and m/z values and normalized intensity values, with no missing/NaN entries in the matrix passed downstream.
- Feature-detection yield on the same input should exceed comparators (e.g. mean 11,442 signals for mzLearn vs 7,100 XCMS and 4,655 ASARI across 15 public datasets).
- Downstream pre-training must succeed: the resulting 2,736-feature matrix should load via pandas.read_csv(index_col=0) as float CSVs of shape [n_samples, 2736].

## Limitations

- Iterative learning increases computational time (up to 2.5 days on a single machine); parallel processing on multiple compute nodes is not yet implemented.
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, so synthetic QC is the only applicable normalization on such data.
- The drift-corrected output is limited to the ion MS1 level; it does not perform de-isotoping or MS/MS identification, and no MS/MS integration into pretraining is implemented.
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments.
- Generalizability across multiple fine-tuning datasets is untested; the current pre-trained drift model is specific to HILIC positive mode.

## Evidence

- [results] synthetic QC normalization achieved 17% RSD in hidden QC samples, comparable to 16% RSD when normalizing with experimental QC samples.: [evidence span withheld — anchor status: not_found]
- [readme] without requiring input parameters from the user ... iteratively learning signal characteristics: [evidence span withheld — anchor status: external_doc]
- [readme] mzLearn autonomously refines signal detection, correcting for retention time (rt) and intensity drifts caused by batch effects and run order.: [evidence span withheld — anchor status: external_doc]
- [readme] No prior knowledge or QC samples are required.: [evidence span withheld — anchor status: external_doc]
- [readme] Output: A two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples.: [evidence span withheld — anchor status: not_found]
