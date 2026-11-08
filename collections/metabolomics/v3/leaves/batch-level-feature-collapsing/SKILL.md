---
name: batch-level-feature-collapsing
description: Use when LC/MS metabolomics samples were run in multiple batches, within-batch
  feature tables already exist per batch, and the goal is to collapse them into batch-level
  matrices for cross-batch alignment and weak signal recovery.
when_to_use_negative:
- Input is a single batch or all samples can be processed in one group — batch-level
  collapsing and cross-batch tolerances are unnecessary.
- Within-batch feature tables do not yet exist — run the first stage (cdf.to.ftr peak
  detection and quantification) first.
- The goal is post hoc intensity-only batch correction of an already aligned feature
  table — the paper notes such methods cannot fix misalignment or incomplete weak
  signal recovery.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: apLCMS
  role: Provides feature.align for within-batch collapsing and cross-batch alignment
    and recover.weaker for weak signal recovery across batches; the batch-aware preprocessing
    method is implemented in the apLCMS package.
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Runtime environment in which the apLCMS batch-aware preprocessing workflow
    (feature.align, recover.weaker) is executed.
- name: foreach
  role: Dependency of apLCMS used to parallelize the per-batch and per-file processing
    loops.
- name: doParallel
  role: Dependency of apLCMS supplying the parallel backend for foreach loops over
    batches and raw files.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:365f856a5b886fad4dbfc1276d4f1205763cc02cb12d75af9758551e638cf8ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  - doParallel
  - foreach
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# batch-level-feature-collapsing

## Summary

Second stage of the apLCMS two-stage multi-batch LC/MS preprocessing: within-batch feature tables produced by cdf.to.ftr are collapsed into one batch-level feature matrix per batch (m/z, mean RT, mean intensity) so that cross-batch alignment and weak signal recovery can be performed on stable batch-level features rather than raw per-file peaks.

## When to use

Use when LC/MS metabolomics samples were run in multiple batches, within-batch feature tables already exist per batch, and the goal is to collapse them into batch-level matrices for cross-batch alignment and weak signal recovery.

## When NOT to use

- Input is a single batch or all samples can be processed in one group — batch-level collapsing and cross-batch tolerances are unnecessary.
- Within-batch feature tables do not yet exist — run the first stage (cdf.to.ftr peak detection and quantification) first.
- The goal is post hoc intensity-only batch correction of an already aligned feature table — the paper notes such methods cannot fix misalignment or incomplete weak signal recovery.

## Inputs

- Batch index table (info) mapping each raw data file to its batch
- Within-batch feature tables produced per batch by cdf.to.ftr
- Within-batch alignment parameters (min.exp, mz.tol, chr.tol, max.align.mz.diff)
- Cross-batch alignment tolerances (batch.align.mz.tol, batch.align.chr.tol)
- Weak signal recovery parameters (recover.mz.range, recover.chr.range, use.observed.range, recover.min.count, intensity.weighted)

## Outputs

- One batch-level feature matrix per batch (m/z, average RT, average intensity)
- Cross-batch aligned feature table with m/z, RT, and per-sample recovered intensities (e.g. aligned_batch_features.csv)

## How to apply

Load the batch index (info) mapping each raw data file to its batch together with the within-batch feature tables produced per batch by cdf.to.ftr. For each batch, run apLCMS feature.align with the within-batch settings (min.exp, mz.tol, chr.tol, max.align.mz.diff) to correct retention times within the batch and collapse features into a single batch-level matrix that stores each feature's m/z, its average RT, and its average intensity across the batch. Then align the batch-level matrices across all batches with feature.align using wider cross-batch tolerances batch.align.mz.tol and batch.align.chr.tol so corresponding features merge despite between-batch RT drift. For every aligned batch-level feature, run recover.weaker on each file's original profile (recover.mz.range, recover.chr.range, use.observed.range, recover.min.count, intensity.weighted) to quantify the feature in samples where the signal was too weak for within-batch detection. Finally assemble m/z, RT, and per-sample recovered intensities into one cross-batch aligned feature table. The rationale, per the paper, is that treating all samples as a single group causes peak alignment errors that post hoc batch effect correction cannot fix.

## Related tools

- **apLCMS** (Provides feature.align for within-batch collapsing and cross-batch alignment and recover.weaker for weak signal recovery across batches; the batch-aware preprocessing method is implemented in the apLCMS package.) — https://github.com/tianwei-yu/apLCMS
- **R** (Runtime environment in which the apLCMS batch-aware preprocessing workflow (feature.align, recover.weaker) is executed.)
- **foreach** (Dependency of apLCMS used to parallelize the per-batch and per-file processing loops.)
- **doParallel** (Dependency of apLCMS supplying the parallel backend for foreach loops over batches and raw files.)

## Examples

```
aligned <- feature.align(batch.ftrs, min.exp=min.exp, mz.tol=mz.tol, chr.tol=chr.tol, max.align.mz.diff=max.align.mz.diff)
```

## Evaluation signals

- aligned_batch_features.csv exists and each aligned row carries an m/z, a retention time, and recovered intensities for the samples.
- No feature is lost between the batch-level alignment and the final recovered table (row count invariant from cross-batch alignment to output).
- Within-batch RTs are corrected (feature.align with within-batch tolerances) before any cross-batch comparison is performed.
- Each aligned feature's recovered intensity set covers samples in all batches, including those where the feature was too weak for within-batch detection.
- Downstream consistency checks: the paper reports the new method yielded feature tables with better consistency and better downstream analysis results than single-group preprocessing.

## Limitations

- Traditional preprocessing treating all samples as a single group can cause peak alignment errors that post hoc batch effect correction methods cannot correct — hence this skill must be applied during preprocessing, not after.
- Post hoc batch effect removal only adjusts signal intensity; it cannot fix feature misalignment across batches or incomplete weak signal recovery, so collapsing must precede any such correction.
- Cross-batch tolerances (batch.align.mz.tol, batch.align.chr.tol) must be wider than within-batch tolerances; mis-set tolerances risk merging non-corresponding features or failing to merge true counterparts.

## Evidence

- [discussion] Two-stage approach for LC/MS metabolomics data generated in multiple batches: "We presented a two-stage approach for LC/MS metabolomics data generated in multiple batches."
- [intro] preprocessing steps enumerated: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
- [intro] limitation of post hoc correction: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features"
- [abstract] better consistency finding: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [discussion] apLCMS package availability: "The package can be downloaded at https://github.com/tianwei-yu/apLCMS."
