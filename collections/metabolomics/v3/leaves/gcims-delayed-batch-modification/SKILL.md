---
name: gcims-delayed-batch-modification
description: Use when all samples in a GCIMSDataset must be transformed identically
  in batch (e.g. cutting trailing drift-time points) and you want the modification
  applied lazily via DelayedOperation rather than editing each sample's matrix one
  by one.
when_to_use_negative:
- When you only need to transform a single sample outside the context of a GCIMSDataset
  — the delayed-operation batch mechanism is tied to dataset-level realize() processing
- When the samples have already been modified in place and no lazy, deferred application
  of the transformation is required
- When you need immediate intermediate results from the per-sample function during
  interactive exploration; realize() defers computation until the delayed operation
  is triggered
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCIMS
  role: R package providing GCIMSDataset, DelayedOperation, realize(), and the fun_extract/fun_aggregate
    hooks used for the batch modification
  repo: https://github.com/sipss/GCIMS
- name: BiocParallel
  role: Loaded alongside GCIMS in the vignette to support parallel execution of the
    sample-processing pipeline
- name: R
  role: Language in which the delayed batch modification workflow is scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.chemolab.2023.104938
    title: 'GCIMS: An R package for untargeted gas chromatography – Ion mobility spectrometry
      data processing'
  source_content_hash: sha256:aca29bdfa57a2b6f82c66bf63622a38126b8de4a33f810334c3a370dd918e9ad
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/sipss/GCIMS
  repo_url: https://github.com/sipss/GCIMS
  tools:
  - BiocParallel
  - GCIMS
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gcims-delayed-batch-modification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A GCIMS-package pattern for modifying every sample in a GCIMSDataset in one batch: the per-sample transformation is wrapped in a delayed operation that is stored on the dataset and only computed during realize(), after which per-sample summaries are extracted and aggregated (e.g. into a cutDrift data frame in x$userData).

## When to use

Use when all samples in a GCIMSDataset must be transformed identically in batch (e.g. cutting trailing drift-time points) and you want the modification applied lazily via DelayedOperation rather than editing each sample's matrix one by one.

## When NOT to use

- When you only need to transform a single sample outside the context of a GCIMSDataset — the delayed-operation batch mechanism is tied to dataset-level realize() processing
- When the samples have already been modified in place and no lazy, deferred application of the transformation is required
- When you need immediate intermediate results from the per-sample function during interactive exploration; realize() defers computation until the delayed operation is triggered

## Inputs

- GCIMSDataset containing the samples to modify (e.g. the threeketones demo dataset)
- Per-sample modification function, e.g. one that trims trailing drift-time points to cut drift
- DelayedOperation wrapper carrying the per-sample function
- fun_extract function producing one extracted object per sample
- fun_aggregate function taking (GCIMSDataset, extracted list) and returning a summary data frame

## Outputs

- Batch-modified GCIMSDataset in which every sample has had the delayed operation applied
- Updated drift-time vectors per sample, changed consistently with the per-sample function
- Per-sample aggregated summary data frame (e.g. cutDrift) with one row per sample, stored in x$userData$cutDrift

## How to apply

Prepare a GCIMSDataset containing the demo samples and write a per-sample function that performs the modification — for example removing trailing drift-time points to cut drift. Wrap this function in a DelayedOperation and append it to the dataset instead of calling it directly, so the transformation is deferred rather than applied eagerly. Realize the dataset: during realize(), per-sample information is extracted into a list with one entry per sample, and the transformation is computed for every sample. To inspect the effect, extract each sample's drift-time vector before and after the modification and confirm the vectors changed as expected. Finally, supply the fun_extract and fun_aggregate hooks: the extraction function collects one object per sample, and the aggregation function — called with the GCIMSDataset as first argument and the extracted list as second argument — converts them into a per-sample summary data frame such as cutDrift, stored in the dataset via x$userData$cutDrift, with one row per sample.

## Related tools

- **GCIMS** (R package providing GCIMSDataset, DelayedOperation, realize(), and the fun_extract/fun_aggregate hooks used for the batch modification) — https://github.com/sipss/GCIMS
- **BiocParallel** (Loaded alongside GCIMS in the vignette to support parallel execution of the sample-processing pipeline)
- **R** (Language in which the delayed batch modification workflow is scripted)

## Examples

```
dataset <- appendDelayedOperation(dataset, cutDriftFn); dataset <- realize(dataset); cutDrift <- dataset$userData$cutDrift
```

## Evaluation signals

- After realize(), every sample in the dataset has been transformed — no sample retains the original unmodified drift-time content
- The aggregated cutDrift data frame contains exactly one row per sample, with per-sample summary values
- Extracting each sample's drift-time vector before and after the modification shows the vectors changed consistently with the applied per-sample function (e.g. trailing drift-time points removed)
- x$userData$cutDrift is populated in the dataset after aggregation
- The transformation was not applied at definition time but only when realize() was triggered (lazy evaluation behaviour)

## Limitations

- The modification is deferred until realize(); the samples are not changed when the DelayedOperation is appended, which can surprise users expecting in-place edits
- Aggregation writes the summary into x$userData (e.g. cutDrift), so the aggregated result lives on the dataset rather than as a standalone return value
- The vignette-specific example (cutting drift) illustrates the mechanism; other per-sample transformations must still be implemented as per-sample functions compatible with the DelayedOperation interface

## Evidence

- [intro] Batch modification via delayed operations on GCIMSDataset: "In this vignette we show how samples can be modified in a batch, using delayed operations."
- [readme] realize() extracts per-sample info and calls aggregation with dataset + list: [evidence span withheld — anchor status: not_found]
- [results] Aggregated summary stored in x$userData$cutDrift: [evidence span withheld — anchor status: not_found]
- [readme] GCIMS package scope: preprocessing pipeline for GC-IMS samples: "GCIMS is an R package implementing a preprocessing pipeline for Gas Chromatography – Ion Mobility Spectrometry samples."
- [readme] Installation from the sipss/GCIMS GitHub repository: [evidence span withheld — anchor status: external_doc]
