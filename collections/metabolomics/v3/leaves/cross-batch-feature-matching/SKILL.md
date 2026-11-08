---
name: cross-batch-feature-matching
description: Use when LC/MS metabolomics raw data were acquired in multiple batches
  and you need cross-batch aligned feature tables with recovered weak signals, rather
  than aligning all samples as one group or merging pre-aligned per-batch tables.
when_to_use_negative:
- Input is already a fully aligned cross-batch feature table — post hoc batch effect
  correction methods are the appropriate choice since this skill works at the preprocessing
  stage
- All samples were processed as a single acquisition batch, so there is no batch index
  or per-batch feature structure to align across
- Only one batch exists and no weak signal recovery across batches is needed
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: apLCMS
  role: Provides feature.align for within-batch and cross-batch matrix alignment and
    recover.weaker for quantifying weak signals in each file's original profile
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Runtime environment in which the apLCMS batch-aware two-stage alignment is
    executed
- name: foreach
  role: Parallel iteration over files/batches during within-batch matrix construction
    and weak signal recovery
- name: doParallel
  role: Parallel backend for the foreach loops used in batch alignment and recovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:2d34e5bc60181054e2b87cc8b6a39fb7fb4db597e730ceea545a2998482ee053
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  - doParallel
  - foreach
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-batch-feature-matching

## Summary

Second stage of the two-stage apLCMS workflow: collapse within-batch feature tables into batch-level feature matrices (per-feature m/z, average RT, average intensity), align them across batches with wider tolerances, and map aligned features back to original within-batch features for weak signal recovery.

## When to use

Use when LC/MS metabolomics raw data were acquired in multiple batches and you need cross-batch aligned feature tables with recovered weak signals, rather than aligning all samples as one group or merging pre-aligned per-batch tables.

## When NOT to use

- Input is already a fully aligned cross-batch feature table — post hoc batch effect correction methods are the appropriate choice since this skill works at the preprocessing stage
- All samples were processed as a single acquisition batch, so there is no batch index or per-batch feature structure to align across
- Only one batch exists and no weak signal recovery across batches is needed

## Inputs

- Batch index (info) mapping each raw data file to its batch
- Within-batch feature tables per batch previously produced by cdf.to.ftr
- Within-batch tolerance parameters (min.exp, mz.tol, chr.tol, max.align.mz.diff)
- Cross-batch tolerance parameters (batch.align.mz.tol, batch.align.chr.tol)
- Weak recovery parameters (recover.mz.range, recover.chr.range, use.observed.range, recover.min.count, intensity.weighted)

## Outputs

- One batch-level feature matrix per batch (m/z, mean RT, mean intensity)
- A single cross-batch aligned feature table with m/z, RT, and per-sample recovered intensities (aligned_batch_features.csv)

## How to apply

Load the batch index (info) mapping each raw file to its batch together with the within-batch feature tables produced per batch by cdf.to.ftr. For each batch, collapse the within-batch feature tables into one batch-level feature matrix — keep each feature's m/z and take the average RT and average intensity — by running apLCMS feature.align within the batch (min.exp, mz.tol, chr.tol, max.align.mz.diff) so RT correction happens within batch before any cross-batch comparison. Then align the batch-level matrices across all batches with feature.align using wider cross-batch tolerances (batch.align.mz.tol, batch.align.chr.tol) to merge corresponding features. For every aligned batch-level feature, run recover.weaker on each file's original profile (recover.mz.range, recover.chr.range, use.observed.range, recover.min.count, intensity.weighted) to quantify the feature in samples where it was too weak for within-batch detection. Assemble m/z, RT, and per-sample recovered intensities into a single cross-batch aligned feature table; this two-stage design fixes alignment errors that post hoc batch effect correction cannot correct.

## Related tools

- **apLCMS** (Provides feature.align for within-batch and cross-batch matrix alignment and recover.weaker for quantifying weak signals in each file's original profile) — https://github.com/tianwei-yu/apLCMS
- **R** (Runtime environment in which the apLCMS batch-aware two-stage alignment is executed)
- **foreach** (Parallel iteration over files/batches during within-batch matrix construction and weak signal recovery)
- **doParallel** (Parallel backend for the foreach loops used in batch alignment and recovery)

## Examples

```
aligned <- adjust.time(batches, has.isotope=0, mz.tol=batch.align.mz.tol, chr.tol=batch.align.chr.tol); final.ftrs <- feature.align(recover.weaker(aligned$aligned.ftrs, files), min.exp=1, mz.tol=batch.align.mz.tol, chr.tol=batch.align.chr.tol)
```

## Evaluation signals

- aligned_batch_features.csv exists and each aligned row carries an m/z, a retention time, and recovered intensities for the samples
- No feature is lost between the batch-level alignment step and the recovered table (row count preserved from cross-batch feature.align output to final table)
- Within-batch feature.align used narrower tolerances (mz.tol, chr.tol) than the cross-batch call (batch.align.mz.tol, batch.align.chr.tol)
- Recovered intensity table has one column per sample file in info, with every batch represented
- Feature detection/quantification consistency across batches improves versus single-group alignment (down-stream analysis results)

## Limitations

- Post hoc batch effect correction applied after standard preprocessing can only adjust signal intensity; it cannot fix feature misalignment across batches or incomplete weak signal recovery from the original data
- No explicit limitations of the new method are discussed in the article
- No apLCMS package version is stated in the context

## Evidence

- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [discussion] two-stage approach: "We presented a two-stage approach for LC/MS metabolomics data generated in multiple batches."
- [intro] workflow stages: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
- [methods] recover.weaker parameters: "`proc.cdf(filename, min.pres, min.run, tol, baseline.correct, baseline.correct.noise.percentile, do.plot, intensity.weighted)`"
- [methods] cdf.to.ftr stage one: "`cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model`"
