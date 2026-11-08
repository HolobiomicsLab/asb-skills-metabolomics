---
name: lcms-peak-detection-preprocessing
description: Use when raw LC/MS profile data (e.g. NetCDF/CDF files from Metabolomics
  Workbench) were acquired in multiple batches and consistent feature detection/quantification
  across batches is required before downstream analysis.
when_to_use_negative:
- Input is already an aligned feature table — post hoc batch-effect correction methods,
  not preprocessing-stage correction, apply (and note they cannot fix misalignment
  or incomplete weak-signal recovery)
- All samples were acquired in a single batch — single-group apLCMS preprocessing
  without batch splitting is sufficient
- Only signal-intensity batch differences are of concern and feature alignment across
  batches is already trusted
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: apLCMS
  role: Core R package implementing the two-stage batch-aware preprocessing workflow
    (cdf.to.ftr, proc.cdf, prof.to.features, feature.align, two.step.hybrid entry
    point)
  repo: https://github.com/tianwei-yu/apLCMS
- name: mzR
  role: Dependency for reading raw LC/MS profile files (CDF) into apLCMS
  repo: https://github.com/tianwei-yu/apLCMS
- name: doParallel
  role: Parallel backend dependency enabling multi-node (n.nodes) preprocessing runs
  repo: https://github.com/tianwei-yu/apLCMS
- name: foreach
  role: Dependency used for parallel iteration over files/threshold sweeps in the
    apLCMS workflow
  repo: https://github.com/tianwei-yu/apLCMS
- name: splines
  role: Dependency supporting cumulative nonlinear retention-time correction
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Language in which the whole preprocessing and consistency-metric computation
    is executed
  repo: https://github.com/tianwei-yu/apLCMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:683549e09a4699449df8755fedf87de03717fd1fc48c73355a5d22a73da288bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  - doParallel
  - foreach
  - mzR
  - splines
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lcms-peak-detection-preprocessing

## Summary

Two-stage, batch-aware apLCMS preprocessing for multi-batch LC/MS metabolomics raw data: within-batch peak detection/quantification, cumulative nonlinear RT correction, cross-batch feature alignment and weak-signal recovery, yielding more consistent feature tables than single-group preprocessing followed by post hoc batch correction.

## When to use

Use when raw LC/MS profile data (e.g. NetCDF/CDF files from Metabolomics Workbench) were acquired in multiple batches and consistent feature detection/quantification across batches is required before downstream analysis.

## When NOT to use

- Input is already an aligned feature table — post hoc batch-effect correction methods, not preprocessing-stage correction, apply (and note they cannot fix misalignment or incomplete weak-signal recovery)
- All samples were acquired in a single batch — single-group apLCMS preprocessing without batch splitting is sufficient
- Only signal-intensity batch differences are of concern and feature alignment across batches is already trusted

## Inputs

- Raw LC/MS profile files (NetCDF/CDF) for all samples
- Sample annotation ('info') mapping each sample to its acquisition batch and replicate-triplet membership
- Detection threshold sweep values for min.exp
- Preprocessing parameters: min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model, recover.mz.range, recover.chr.range

## Outputs

- Aligned feature matrix (samples x features) per preprocessing method and threshold
- Consistency metrics table (method-by-threshold) of proportion of zeros and aggregated within-triplet CV, e.g. fig3a_3c_consistency_metrics.csv
- Plots of zero proportion and within-triplet CV versus min.exp threshold comparing two-stage vs traditional workflows

## How to apply

Download raw CDF profile files and a sample annotation with batch and replicate-triplet labels as the 'info' input. For a sweep of detection thresholds (min.exp), run traditional single-group preprocessing via cdf.to.ftr, which chains proc.cdf, prof.to.features and feature.align on all samples pooled together. In parallel, run the two-stage batch-aware workflow: preprocess each batch separately, then apply cumulative nonlinear retention-time correction, cross-batch feature alignment, and weak-signal recovery across batches controlled by recover.mz.range / recover.chr.range. Key proc.cdf parameters include min.pres, min.run, mz.tol, and baseline.correct.noise.percentile; the method depends on mzR, splines, doParallel, foreach and MASS. Rationale: single-group alignment across batches can cause alignment errors that post hoc batch-effect correction cannot fix, whereas addressing batch effects at the preprocessing stage improves peak detection, alignment and quantification. From each aligned feature matrix compute the proportion of zeros and, per replicate triplet, the coefficient of variation of feature intensities to judge consistency (lower CV and fewer zeros = better).

## Related tools

- **apLCMS** (Core R package implementing the two-stage batch-aware preprocessing workflow (cdf.to.ftr, proc.cdf, prof.to.features, feature.align, two.step.hybrid entry point)) — https://github.com/tianwei-yu/apLCMS
- **mzR** (Dependency for reading raw LC/MS profile files (CDF) into apLCMS) — https://github.com/tianwei-yu/apLCMS
- **doParallel** (Parallel backend dependency enabling multi-node (n.nodes) preprocessing runs) — https://github.com/tianwei-yu/apLCMS
- **foreach** (Dependency used for parallel iteration over files/threshold sweeps in the apLCMS workflow) — https://github.com/tianwei-yu/apLCMS
- **splines** (Dependency supporting cumulative nonlinear retention-time correction) — https://github.com/tianwei-yu/apLCMS
- **R** (Language in which the whole preprocessing and consistency-metric computation is executed) — https://github.com/tianwei-yu/apLCMS

## Examples

```
features <- cdf.to.ftr(folder='st000868_cdf/', file.pattern='.cdf', n.nodes=8, min.exp=20, min.pres=0.5, min.run=12, mz.tol=1e-5, baseline.correct.noise.percentile=0.05)
```

## Evaluation signals

- Two-stage workflow should yield lower within-cell-type-per-batch CVs of replicate-triplet intensities than traditional apLCMS and XCMS across the min.exp sweep
- Two-stage aligned feature matrix should show a lower (or comparable) proportion of zeros across all samples versus single-group preprocessing
- Every raw sample in the batch annotation must appear in the final aligned feature matrix; no samples silently dropped
- Metrics assemble cleanly into a method-by-threshold table (fig3a_3c_consistency_metrics.csv) with a value for every (method, min.exp) combination
- Trends of zero proportion and within-triplet CV versus threshold should be smooth/monotone-ish across the sweep, not erratic per method

## Limitations

- Post hoc batch-effect correction methods can only adjust signal intensity; they cannot fix misalignment of features across batches or incomplete weak-signal recovery — hence correction must happen at preprocessing stage
- No apLCMS package version is stated in the article or repository documentation
- Requires reliable batch/replicate annotation ('info'); the method directly uses batch information, so mislabeled batches undermine the approach

## Evidence

- [abstract] we developed a new approach that address the batch effect issue in the preprocessing stage, resulting in better peak detection, alignment and quantification: "we developed a new approach that address the batch effect issue in the preprocessing stage, resulting in better peak detection, alignment and quantification"
- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [methods] cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model: "cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [intro] including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
