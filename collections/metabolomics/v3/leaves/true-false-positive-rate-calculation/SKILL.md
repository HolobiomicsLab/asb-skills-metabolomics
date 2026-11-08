---
name: true-false-positive-rate-calculation
description: Use when comparing peak-detection/alignment tools on the same LC/MS datasets
  and you need to quantify which detections are real versus noise, e.g. reporting
  TPR 89.0%/77.4%/49.6% for mzLearn/XCMS/ASARI.
when_to_use_negative:
- Input is already a peak/feature table rather than raw mzML — this skill compares
  tool outputs, not raw-file processing
- There is no ground-truth labeling (peak-shape based or otherwise) available — TPR/FPR
  cannot be computed without it
- The comparison targets downstream tasks (embedding, survival prediction) rather
  than signal detection quality
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzLearn
  role: Signal-detection tool being benchmarked; runs directly from mzML without user-set
    parameters, producing the detected MS1 signal set scored for TPR/FPR
  repo: http://mzlearn.com/
- name: XCMS
  role: Comparator peak-detection and alignment tool whose detected signals are scored
    against the same ground-truth labels
- name: ASARI
  role: Comparator feature-detection tool whose detected signals are scored against
    the same ground-truth labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:f2f5ba45b9c813b1e203b24623c95f79a73ad8c1944633afbe48afd6f24a0784
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
  - mzLearn
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# true-false-positive-rate-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes true positive rate (TPR) and false positive rate (FPR) for LC/MS signal-detection algorithms (mzLearn, XCMS, ASARI) by scoring each detected MS1 signal against a peak-shape-based ground-truth label set across benchmarking datasets.

## When to use

Use when comparing peak-detection/alignment tools on the same LC/MS datasets and you need to quantify which detections are real versus noise, e.g. reporting TPR 89.0%/77.4%/49.6% for mzLearn/XCMS/ASARI.

## When NOT to use

- Input is already a peak/feature table rather than raw mzML — this skill compares tool outputs, not raw-file processing
- There is no ground-truth labeling (peak-shape based or otherwise) available — TPR/FPR cannot be computed without it
- The comparison targets downstream tasks (embedding, survival prediction) rather than signal detection quality

## Inputs

- Detected MS1 signal/feature lists per tool per dataset (peak tables with m/z, median rt, intensities)
- Peak-shape-based ground-truth label set (manually labeled true/false peaks) for each benchmark dataset
- The 15 public targeted-metabolomics benchmark datasets as mzML files

## Outputs

- Per-tool-per-dataset TPR and FPR values
- Average detected signal counts per tool (e.g. 11,442 / 7,100 / 4,655)
- benchmark_results.csv — single comparison table of counts, TPR, FPR for mzLearn, XCMS, ASARI

## How to apply

Process each benchmark mzML dataset with every tool under comparison (mzLearn, XCMS, ASARI), then tabulate detected MS1 signals per dataset per tool. Score each detected feature against ground-truth labels derived from peak-shape-based manual labeling to classify it as true or false positive. Compute TPR as the fraction of true signals recovered and FPR as the fraction of detections that are noise (mzLearn 12.5%, XCMS 17.3%, ASARI 18.8% on the 15-dataset benchmark). Compile counts, TPR and FPR for all tools into a single comparison table (benchmark_results.csv) and verify against expected averages: 11,442 / 7,100 / 4,655 signals and TPR 89.0% / 77.4% / 49.6% for mzLearn / XCMS / ASARI.

## Related tools

- **mzLearn** (Signal-detection tool being benchmarked; runs directly from mzML without user-set parameters, producing the detected MS1 signal set scored for TPR/FPR) — http://mzlearn.com/
- **XCMS** (Comparator peak-detection and alignment tool whose detected signals are scored against the same ground-truth labels)
- **ASARI** (Comparator feature-detection tool whose detected signals are scored against the same ground-truth labels)

## Examples

```
python -m mz_embed.pretrain.get_pretrain_encoder --model_dir /path/to/out_pretrain --data_location /path/to/pretrain_csvs --output_dir /path/to/encoder_out  # then score detected peaks vs ground-truth labels to compute TPR/FPR
```

## Evaluation signals

- Reproduced average signal counts match 11,442 (mzLearn), 7,100 (XCMS), 4,655 (ASARI)
- Reproduced TPR matches 89.0% (mzLearn), 77.4% (XCMS), 49.6% (ASARI)
- Reproduced FPR matches 12.5% (mzLearn), 17.3% (XCMS), 18.8% (ASARI)
- Each detected feature maps to a valid ground-truth label (true/false) with no unclassified peaks
- benchmark_results.csv contains one row per tool per dataset with all three metrics populated

## Limitations

- Ground-truth labels are peak-shape-based and generated via expert review (manual peak-shape labeling), introducing subjective assessment
- mzLearn results are limited to the ion MS1 level; the comparison does not evaluate downstream de-isotoping or MS/MS identification
- Iterative learning in mzLearn can take up to 2.5 days on a single machine, and parallel processing on multiple compute nodes is not yet implemented

## Evidence

- [methods] Workflow card: verify reproduced counts and rates, and submit FPR values to expert review: [evidence span withheld — anchor status: not_found]
- [methods] Workflow card: assess features against ground-truth labels to compute TPR and FPR: [evidence span withheld — anchor status: not_found]
- [readme] README: mzLearn is zero-parameter and iteratively refines signal detection: [evidence span withheld — anchor status: not_found]
