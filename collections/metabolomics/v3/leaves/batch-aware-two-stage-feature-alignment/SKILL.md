---
name: batch-aware-two-stage-feature-alignment
description: Use when raw LC/MS profile files (e.g. CDF) from one study were acquired
  in multiple batches and you need consistent feature detection/quantification across
  batches before downstream batch effect correction.
when_to_use_negative:
- The input is already a feature table or aligned peak matrix — post hoc batch effect
  correction, not preprocessing-stage correction, is then the only option, and this
  skill cannot retroactively fix misalignment
- The data were all acquired in a single batch, so traditional single-group apLCMS
  preprocessing via cdf.to.ftr suffices
- You need only downstream statistical batch effect removal (intensity adjustment),
  which the paper notes can only adjust signal intensity and cannot fix feature misalignment
  or incomplete weak signal recovery
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: apLCMS
  role: R package implementing the two-stage batch-aware preprocessing (two.step.hybrid,
    cdf.to.ftr, proc.cdf, prof.to.features, feature.align)
  repo: https://github.com/tianwei-yu/apLCMS
- name: mzR
  role: Dependency for reading raw mass-spectrometry profile files in R
  repo: https://github.com/tianwei-yu/apLCMS
- name: splines
  role: Dependency supporting cumulative nonlinear retention-time correction across
    batches
- name: doParallel
  role: Dependency for parallelized preprocessing across files/nodes
- name: foreach
  role: Dependency for parallel loop execution in the apLCMS pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:9985fadc84a39d223638acdd7162709d09caee9906facdc2ca2f8ba66a33fc90
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# batch-aware-two-stage-feature-alignment

## Summary

A two-stage apLCMS preprocessing workflow for multi-batch LC/MS metabolomics data that uses batch information during peak detection, RT correction, feature alignment, and weak signal recovery to improve feature-table consistency, rather than treating all samples as a single group or batching independently.

## When to use

Use when raw LC/MS profile files (e.g. CDF) from one study were acquired in multiple batches and you need consistent feature detection/quantification across batches before downstream batch effect correction.

## When NOT to use

- The input is already a feature table or aligned peak matrix — post hoc batch effect correction, not preprocessing-stage correction, is then the only option, and this skill cannot retroactively fix misalignment
- The data were all acquired in a single batch, so traditional single-group apLCMS preprocessing via cdf.to.ftr suffices
- You need only downstream statistical batch effect removal (intensity adjustment), which the paper notes can only adjust signal intensity and cannot fix feature misalignment or incomplete weak signal recovery

## Inputs

- Raw LC/MS profile data files (NetCDF/CDF) organized by acquisition batch
- Sample annotation ('info') mapping files to batches and replicate triplets
- Parameter sweep values for min.exp detection threshold
- Recovery-window settings: recover.mz.range, recover.chr.range

## Outputs

- Batch-aligned feature intensity matrix (features x samples) with reduced proportion of zeros
- Per-feature intensities supporting within-triplet CV computation
- Method-by-threshold consistency metrics table (e.g. fig3a_3c_consistency_metrics.csv) and zero-proportion / CV-versus-threshold plots

## How to apply

Annotate each raw file with its batch and replicate structure in an 'info' data frame, then call the two-stage entry point (two.step.hybrid) instead of cdf.to.ftr on pooled files. Stage one preprocesses each batch separately (proc.cdf, prof.to.features via cdf.to.ftr semantics) with a swept min.exp detection threshold; stage two performs cumulative nonlinear retention-time correction across batches (splines-based), cross-batch feature alignment, and weak signal recovery controlled by recover.mz.range / recover.chr.range. Rationale: single-group alignment across batches causes peak alignment errors that post hoc batch-effect correction cannot fix, because such methods only adjust signal intensity and cannot repair misalignment or incomplete weak-signal recovery. Judge success by computing the proportion of zeros in the final aligned feature matrix and within-replicate-triplet coefficients of variation (as on ST000868), expecting lower values than traditional apLCMS or XCMS at matched thresholds.

## Related tools

- **apLCMS** (R package implementing the two-stage batch-aware preprocessing (two.step.hybrid, cdf.to.ftr, proc.cdf, prof.to.features, feature.align)) — https://github.com/tianwei-yu/apLCMS
- **mzR** (Dependency for reading raw mass-spectrometry profile files in R) — https://github.com/tianwei-yu/apLCMS
- **splines** (Dependency supporting cumulative nonlinear retention-time correction across batches)
- **doParallel** (Dependency for parallelized preprocessing across files/nodes)
- **foreach** (Dependency for parallel loop execution in the apLCMS pipeline)

## Examples

```
result <- two.step.hybrid(folder = 'st000868_cdf/', file.pattern = '.CDF', info = batch_info, min.exp = c(20, 50, 100), min.pres = 0.5, min.run = 12, mz.tol = 1e-5, recover.mz.range = 1e-5, recover.chr.range = 10)
```

## Evaluation signals

- Proportion of zeros across all samples in the final aligned feature matrix is lower than traditional single-group apLCMS and XCMS at matched min.exp thresholds
- Within-replicate-triplet coefficient of variation of feature intensities (within-cell-type-per-batch CV) is lower than comparison methods after adjusting for cell type and batch
- Feature tables show better consistency in detected features across batches (better peak detection, alignment and quantification)
- Recovery settings recover.mz.range / recover.chr.range are explicitly set and weak signals missing in one batch are recovered from the original raw data
- Each batch was preprocessed separately before cumulative nonlinear RT correction and cross-batch alignment, rather than pooling all files into a single cdf.to.ftr run

## Limitations

- Post hoc batch effect correction methods can only adjust signal intensity — they cannot fix misaligned features or incomplete weak signal recovery, so errors from single-group preprocessing are irreversible later
- No specific apLCMS package version is stated in the article or repo context
- No explicit limitations of the new two-stage method are discussed in the article's discussion section
- The supplementary information section content is not available in the extracted context

## Evidence

- [abstract] we developed a new approach that address the batch effect issue in the preprocessing stage, resulting in better peak detection, alignment and quantification: "we developed a new approach that address the batch effect issue in the preprocessing stage, resulting in better peak detection, alignment and quantification"
- [abstract] post hoc correction cannot fix alignment errors: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [intro] post hoc methods only adjust intensity: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [intro] pipeline steps: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
- [methods] cdf.to.ftr signature: "`cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
- [discussion] two-stage presentation: "We presented a two-stage approach for LC/MS metabolomics data generated in multiple batches."
