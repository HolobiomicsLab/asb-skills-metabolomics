---
name: cross-batch-peak-alignment
description: Use when LC/MS profiles are acquired in multiple batches and RT drift
  occurs both within batches and between batches, so that treating all samples as
  one group or preprocessing each batch separately causes peak misalignment.
when_to_use_negative:
- Input is already a batch-corrected feature table — post hoc batch effect correction
  can only adjust signal intensity and cannot fix feature misalignment
- Single-batch datasets with no between-batch structure, where the between-batch stage
  g_k is unnecessary
- When only intensity normalization between batches is needed and peak detection/alignment
  are already correct
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: apLCMS
  role: Implements the two-stage batch-aware RT correction and alignment pipeline
    (cdf.to.ftr, proc.cdf, find.tol.time, rm.ridge, adjust.time)
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Runtime environment for the apLCMS package and its dependency stack (splines,
    MASS, rgl, mzR, doParallel, foreach)
  repo: https://github.com/tianwei-yu/apLCMS
- name: splines
  role: Provides splines-based smoothing for fitting the kernel-smoothed nonlinear
    within-batch RT correction curves
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:689b7f86dd8b57a83adf4329297117c8803cdbc0b27837a759ff8784d157733c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  - splines
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-batch-peak-alignment

## Summary

A two-stage retention-time correction and peak alignment procedure for multi-batch LC/MS metabolomics data that composes within-batch kernel-smoothed RT curves with between-batch alignment curves, implemented in the apLCMS R package.

## When to use

Use when LC/MS profiles are acquired in multiple batches and RT drift occurs both within batches and between batches, so that treating all samples as one group or preprocessing each batch separately causes peak misalignment.

## When NOT to use

- Input is already a batch-corrected feature table — post hoc batch effect correction can only adjust signal intensity and cannot fix feature misalignment
- Single-batch datasets with no between-batch structure, where the between-batch stage g_k is unnecessary
- When only intensity normalization between batches is needed and peak detection/alignment are already correct

## Inputs

- Raw LC/MS profile files (e.g. NetCDF/cdf) organized by batch
- Batch labels assigning each sample to a batch and a reference batch
- apLCMS parameters: min.pres, min.run, mz.tol, chr.tol, find.tol.max.d, max.align.mz.diff, aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments

## Outputs

- Batch-aware feature table with m/z, corrected retention time, and intensity per feature per sample
- Fitted within-batch RT curves f_k,j and between-batch correction curves g_k
- Aligned peaks supporting downstream feature alignment and weak signal recovery

## How to apply

Load raw LC/MS profiles with apLCMS (cdf.to.ftr wrapping proc.cdf) to build per-sample feature tables of m/z, retention time, and intensity, then estimate the RT tolerance across samples with find.tol.time (aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments). In stage one, remove ridges (rm.ridge) and fit, within each batch, a splines-based kernel-smoothed nonlinear RT curve f_k,j of each sample against the batch reference sample. In stage two, fit the between-batch correction curve g_k by aligning each batch to the reference batch using the adjust.time alignment logic (mz.tol, chr.tol, find.tol.max.d, max.align.mz.diff). Apply the cumulative correction t_corrected = t - f(t) - g(t - f(t)) to every detected peak in every profile, then proceed to feature alignment and weak signal recovery. Validate by comparing within-batch RT curves (e.g. ST000868) before and after correction to confirm reduced RT drift.

## Related tools

- **apLCMS** (Implements the two-stage batch-aware RT correction and alignment pipeline (cdf.to.ftr, proc.cdf, find.tol.time, rm.ridge, adjust.time)) — https://github.com/tianwei-yu/apLCMS
- **R** (Runtime environment for the apLCMS package and its dependency stack (splines, MASS, rgl, mzR, doParallel, foreach)) — https://github.com/tianwei-yu/apLCMS
- **splines** (Provides splines-based smoothing for fitting the kernel-smoothed nonlinear within-batch RT correction curves)

## Examples

```
library(apLCMS); ftrs <- cdf.to.ftr(folder='raw/', file.pattern='.cdf', n.nodes=4, min.exp=5, min.pres=0.5, min.run=12, mz.tol=1e-5, batch=TRUE, find.tol.max.d=1e-4, max.align.mz.diff=0.01)
```

## Evaluation signals

- Within-batch RT curves from a test study (e.g. ST000868) show visibly reduced drift after correction compared with before
- Peak alignment across batches no longer contains misaligned features that post hoc batch effect correction could not fix
- Feature detection and quantification consistency across batches improves relative to single-group or per-batch preprocessing
- Downstream analyses on the corrected feature tables perform better than on traditionally preprocessed tables

## Limitations

- Post hoc batch effect correction methods applied after preprocessing can only adjust signal intensity; they cannot fix feature misalignment or incomplete weak signal recovery
- Traditional preprocessing that treats all samples as a single group can cause peak alignment errors that cannot be corrected afterwards
- No apLCMS package version is stated in the article; the package must be obtained from the GitHub repository

## Evidence

- [intro] two-stage cumulative RT correction: "we propose a new approach that preprocess the data in a two-stage manner. The method directly uses the batch information to allow optimal within-batch and"
- [intro] post hoc limitation: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [methods] find.tol.time signature: "find.tol.time(mz, chr, lab, num.exp, mz.tol, chr.tol, aver.bin.size, min.bins, max.bins, max.mz.diff, max.num.segments)"
- [abstract] alignment motivation: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [discussion] outcome: "we showed that the new method improved the consistency of feature detection and quantification"
