---
name: batch-reference-curve-fitting
description: Use when LC/MS profiles are generated in multiple batches and peaks show
  RT drift at two levels (within-batch and between-batch); signals include inconsistent
  feature retention times across batches and misaligned peaks after single-group preprocessing.
when_to_use_negative:
- Input is already an aligned feature table from a single-group preprocessing run
  — the article notes post hoc batch effect correction cannot fix alignment errors
  introduced upstream
- Data come from a single batch with no between-batch structure — only the within-batch
  RT curve f is needed, and the two-stage composition adds no benefit
- The goal is only to adjust signal intensities between batches — the article states
  such approaches 'can only adjust signal intensity' and cannot address feature misalignment,
  which is what this skill targets
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: apLCMS
  role: Host platform implementing the two-stage batch-aware preprocessing, including
    proc.cdf peak picking, rm.ridge, find.tol.time, and adjust.time alignment
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Runtime environment for apLCMS; the package Depends on splines and other R
    packages, so load dependencies explicitly
- name: splines
  role: Provides the spline smoothing used to fit the within-batch kernel-smoothed
    RT curves f_k,j
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:6bee19b5394a661ef61c7a091eb8e7270eac51497ad8b67b177d5cb2cd453441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  - splines
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# batch-reference-curve-fitting

## Summary

A two-stage retention time (RT) correction procedure for multi-batch LC/MS metabolomics data: within each batch a kernel-smoothed RT curve is fit against the batch reference sample, and a second between-batch curve aligns each batch to the reference batch; the two fitted deviations are summed into a single per-profile RT correction.

## When to use

Use when LC/MS profiles are generated in multiple batches and peaks show RT drift at two levels (within-batch and between-batch); signals include inconsistent feature retention times across batches and misaligned peaks after single-group preprocessing.

## When NOT to use

- Input is already an aligned feature table from a single-group preprocessing run — the article notes post hoc batch effect correction cannot fix alignment errors introduced upstream
- Data come from a single batch with no between-batch structure — only the within-batch RT curve f is needed, and the two-stage composition adds no benefit
- The goal is only to adjust signal intensities between batches — the article states such approaches 'can only adjust signal intensity' and cannot address feature misalignment, which is what this skill targets

## Inputs

- Raw LC/MS profile files (e.g. NetCDF/cdf, mzML) organized into batches
- Batch labels mapping each sample/profile to its batch
- Feature tables per sample with m/z, retention time, and intensity columns
- apLCMS parameter settings: min.pres, min.run, mz.tol, chr.tol, aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments, find.tol.max.d, max.align.mz.diff

## Outputs

- Batch-aware corrected feature table with aligned retention times across all batches
- Per-sample within-batch RT correction curves f_k,j (spline/kernel fits)
- Per-batch between-batch correction curves g_k fitted by adjust.time alignment
- Per-profile overall RT correction values t - f(t) - g(t - f(t)) applied to every detected peak

## How to apply

Load raw LC/MS profiles (NetCDF/mzML) with apLCMS (load.lcms/proc.cdf via cdf.to.ftr) and build per-sample feature tables of m/z, retention time, and intensity. Estimate RT tolerance across samples with find.tol.time using parameters aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments. Within each batch, first remove ridges with rm.ridge, then fit a kernel-smoothed nonlinear RT curve f_k,j of every sample against the batch reference sample using splines-based smoothing. Fit the between-batch correction curve g_k by aligning each batch to the reference batch using the adjust.time alignment logic (mz.tol, chr.tol, find.tol.max.d, max.align.mz.diff). Apply the composed correction t_corrected = t - f(t) - g(t - f(t)) to every detected peak in every profile; the additive composition matters because the between-batch drift is evaluated on the already within-batch-corrected time scale.

## Related tools

- **apLCMS** (Host platform implementing the two-stage batch-aware preprocessing, including proc.cdf peak picking, rm.ridge, find.tol.time, and adjust.time alignment) — https://github.com/tianwei-yu/apLCMS
- **R** (Runtime environment for apLCMS; the package Depends on splines and other R packages, so load dependencies explicitly)
- **splines** (Provides the spline smoothing used to fit the within-batch kernel-smoothed RT curves f_k,j)

## Examples

```
proc.cdf(filename, min.pres, min.run, tol, baseline.correct, baseline.correct.noise.percentile, do.plot, intensity.weighted)
```

## Evaluation signals

- Compare within-batch RT curves (e.g. ST000868) before and after correction: residual RT drift should be visibly reduced after applying t - f(t) - g(t - f(t))
- Feature detection and quantification consistency across batches should improve relative to single-group or per-batch-preprocessed baselines
- Peak alignment errors that persisted under traditional single-group preprocessing should be resolved in the corrected feature table
- Downstream analysis results (e.g. group separation/association tests) should be at least as good as, and ideally better than, those from uncorrected or post-hoc-only corrected tables
- Each fitted correction curve f_k,j and g_k should capture monotone/smooth RT deviation structure without overfitting; check that residuals after each fit lack systematic ridge structure

## Limitations

- Post-processing batch effect correction methods applied after preprocessing can only adjust signal intensity and cannot fix feature misalignment or incomplete weak signal recovery — correction must happen at the preprocessing stage
- Traditional preprocessing that treats all samples as a single group can create peak alignment errors that cannot be corrected post hoc
- No explicit limitations of the new method are discussed in the article
- No apLCMS package version is stated in the article or repo documentation

## Evidence

- [intro] we propose a new approach that preprocess the data in a two-stage manner. The method directly uses the batch information to allow optimal within-batch and: "we propose a new approach that preprocess the data in a two-stage manner. The method directly uses the batch information to allow optimal within-batch and"
- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [methods] find.tol.time(mz, chr, lab, num.exp, mz.tol, chr.tol, aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments): "`find.tol.time(mz, chr, lab, num.exp, mz.tol, chr.tol, aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments)`"
- [methods] proc.cdf(filename, min.pres, min.run, tol, baseline.correct, baseline.correct.noise.percentile, do.plot, intensity.weighted): "`proc.cdf(filename, min.pres, min.run, tol, baseline.correct, baseline.correct.noise.percentile, do.plot, intensity.weighted)`"
- [intro] including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
