---
name: ms1-signal-detection-pipeline-execution
description: Use when raw LC/MS data in mzML format needs automated MS1 peak detection
  and alignment without user-set parameters or QC samples, or when benchmarking signal
  counts/TP-FP rates against XCMS/ASARI.
when_to_use_negative:
- You need de-isotoped or MS/MS-identified compounds — mzLearn outputs ion MS1 level
  only and does not perform de-isotoping or MS/MS identification.
- Your data is not in mzML format (e.g., already a feature table or proprietary vendor
  format).
- Your goal is downstream modeling (pretraining/fine-tuning a VAE) rather than the
  signal-detection step itself — that is the mzEmbed fine-tuning workflow, not this
  skill.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: mzLearn
  role: Primary zero-parameter MS1 signal-detection and alignment algorithm run directly
    on mzML files; iteratively learns signal characteristics, corrects rt/intensity
    drift, outputs feature table
  repo: http://mzlearn.com/
- name: XCMS
  role: 'Comparison tool: runs its own peak-detection and alignment workflow on the
    same mzML files for TP/FP benchmarking'
- name: ASARI
  role: 'Comparison tool: runs its own peak-detection and alignment workflow on the
    same mzML files for TP/FP benchmarking'
- name: mzEmbed
  role: Repository codebase containing mzLearn and downstream mzEmbed framework; used
    for running and extending the signal-detection pipeline
  repo: https://github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:8ee131235df376f1e2b729e4009673cfdb10bf95db45ba4df6c20926a36702a1
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ms1-signal-detection-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute the mzLearn MS1 signal-detection and alignment pipeline: run raw LC/MS mzML files through the zero-parameter mzLearn algorithm to produce a feature table of median rt/m/z with normalized intensities, then benchmark outputs against XCMS and ASARI. This skill encapsulates the paper's core data-processing workflow, from file ingestion to signal counts and TP/FP evaluation.

## When to use

Use when raw LC/MS data in mzML format needs automated MS1 peak detection and alignment without user-set parameters or QC samples, or when benchmarking signal counts/TP-FP rates against XCMS/ASARI.

## When NOT to use

- You need de-isotoped or MS/MS-identified compounds — mzLearn outputs ion MS1 level only and does not perform de-isotoping or MS/MS identification.
- Your data is not in mzML format (e.g., already a feature table or proprietary vendor format).
- Your goal is downstream modeling (pretraining/fine-tuning a VAE) rather than the signal-detection step itself — that is the mzEmbed fine-tuning workflow, not this skill.

## Inputs

- Raw LC/MS data files in mzML format
- 15 public targeted-metabolomics benchmarking datasets (Supplementary Table 1)
- Peak-shape-based ground-truth labels for TP/FP assessment

## Outputs

- Two-dimensional feature table: median rt and m/z values with normalized intensities across samples (MS1 ion level)
- Per-tool detected-signal counts and averages across datasets
- TP and FP rates for mzLearn vs XCMS vs ASARI (benchmark_results.csv)

## How to apply

1) Retrieve raw LC/MS files as mzML (e.g., the 15 targeted-metabolomics benchmarking datasets enumerated in Supplementary Table 1). 2) Process each dataset with mzLearn, which runs directly from mzML files with zero user-set parameters — no prior knowledge or QC samples are required — and iteratively learns internal thresholds plus intensity/rt drift maps (generating synthetic QC samples). 3) Process the same mzML files with XCMS and ASARI using their respective peak-detection and alignment workflows for comparison. 4) Tabulate detected MS1 signals per tool and compute averages and TP/FP rates against peak-shape-based ground-truth labels; expected reference values are mzLearn 11,442 signals / 89.0% TP / 12.5% FP, XCMS 7,100 / 77.4% / 17.3%, ASARI 4,655 / 49.6% / 18.8%. 5) mzLearn output is a two-dimensional table of detected features defined by median rt and m/z with normalized intensities across samples; outputs are ion-level MS1 only (no de-isotoping or MS/MS identification).

## Related tools

- **mzLearn** (Primary zero-parameter MS1 signal-detection and alignment algorithm run directly on mzML files; iteratively learns signal characteristics, corrects rt/intensity drift, outputs feature table) — http://mzlearn.com/
- **XCMS** (Comparison tool: runs its own peak-detection and alignment workflow on the same mzML files for TP/FP benchmarking)
- **ASARI** (Comparison tool: runs its own peak-detection and alignment workflow on the same mzML files for TP/FP benchmarking)
- **mzEmbed** (Repository codebase containing mzLearn and downstream mzEmbed framework; used for running and extending the signal-detection pipeline) — https://github.com/ReviveMed/mzEmbed

## Examples

```
git clone git@github.com:ReviveMed/mzEmbed.git && cd mzEmbed && pip install -r requirements.txt; python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/mzml_files --output_dir out
```

## Evaluation signals

- Average detected signals across the 15 benchmarking datasets match 11,442 (mzLearn), 7,100 (XCMS), 4,655 (ASARI)
- TP rates match 89.0% / 77.4% / 49.6% and FP rates match 12.5% / 17.3% / 18.8% for mzLearn/XCMS/ASARI respectively
- Feature table is a 2-D matrix with median rt and m/z columns and normalized intensities per sample
- No user-set parameters were supplied and no experimental QC samples were used to estimate drift
- Feature count exceeds the >60%-more-signals claim when compared with existing methods on additional datasets

## Limitations

- Iterative learning increases computational time (up to 2.5 days on a single machine); parallel processing on multiple compute nodes is not yet implemented
- mzLearn outputs are limited to ion MS1 level; no de-isotoping or MS/MS identification, and no MS/MS integration into pretraining
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift; mzLearn mitigates this via synthetic QC but large-scale deployments may still need batch covariates or domain-invariant regularization

## Evidence

- [readme] Output is a 2D table of detected features with median rt/m/z and normalized intensities; scalable to 2,075 files in a single run: [evidence span withheld — anchor status: not_found]
- [readme] Zero-parameter design: no prior knowledge or QC samples required: [evidence span withheld — anchor status: not_found]
- [readme] Scalability of mzLearn to large datasets (e.g., 2,075 files in a single run): [evidence span withheld — anchor status: not_found]
