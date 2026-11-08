---
name: batch-effect-aware-preprocessing
description: Use when LC/MS metabolomics raw profiles (e.g. NetCDF/mzXML) were acquired
  in multiple batches and treating all samples as one group would risk peak alignment
  errors, or batch-wise preprocessing alone would prevent cross-batch weak signal
  recovery.
when_to_use_negative:
- All samples were acquired in a single batch, so within-batch preprocessing alone
  (stage one, cdf.to.ftr) is sufficient and there is no batch structure to exploit.
- The goal is only post-hoc intensity adjustment after a single-group preprocessing
  run — the article notes such approaches 'can only adjust signal intensity' and cannot
  fix cross-batch misalignment or incomplete weak signal recovery.
- Input is already a finished feature table from single-group alignment; re-running
  cannot correct alignment errors made at that stage.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: apLCMS
  role: 'Implements the entire two-stage batch-aware workflow: proc.cdf/cdf.to.ftr
    within-batch processing, feature.align within and across batches, and recover.weaker
    for weak signal recovery'
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Runtime environment for apLCMS; package dependencies (MASS, rgl, mzR, splines,
    doParallel, foreach) must be attached explicitly with library()
- name: foreach
  role: Depends-level package supporting parallelized per-file/per-batch feature extraction
    in the apLCMS workflow
- name: doParallel
  role: Depends-level package providing the parallel backend (n.nodes) for foreach
    loops over raw files in cdf.to.ftr
- name: mzR
  role: Depends-level package for reading raw mass-spectrometry profile files in apLCMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:b7ce632ec5d2ae09929dc8c4bccdcd4c76fae8cd81736dee708aeebd83392e53
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# batch-effect-aware-preprocessing

## Summary

A two-stage apLCMS preprocessing method for LC/MS metabolomics data generated in multiple batches: within-batch peak detection/RT correction/alignment first, then cross-batch alignment with weak signal recovery, addressing batch effects during preprocessing rather than post hoc.

## When to use

Use when LC/MS metabolomics raw profiles (e.g. NetCDF/mzXML) were acquired in multiple batches and treating all samples as one group would risk peak alignment errors, or batch-wise preprocessing alone would prevent cross-batch weak signal recovery.

## When NOT to use

- All samples were acquired in a single batch, so within-batch preprocessing alone (stage one, cdf.to.ftr) is sufficient and there is no batch structure to exploit.
- The goal is only post-hoc intensity adjustment after a single-group preprocessing run — the article notes such approaches 'can only adjust signal intensity' and cannot fix cross-batch misalignment or incomplete weak signal recovery.
- Input is already a finished feature table from single-group alignment; re-running cannot correct alignment errors made at that stage.

## Inputs

- Multi-batch LC/MS raw profile files (NetCDF/mzXML/mzML) with a batch index mapping each file to its batch
- Within-batch feature tables previously produced per batch by cdf.to.ftr
- Parameters: min.exp, min.pres, min.run, mz.tol, chr.tol, max.align.mz.diff, batch.align.mz.tol, batch.align.chr.tol, recover.mz.range, recover.chr.range, use.observed.range, recover.min.count, intensity.weighted

## Outputs

- Per-batch batch-level feature matrices (m/z, average RT, average intensity)
- One cross-batch aligned feature table (aligned_batch_features.csv) with each aligned row carrying an m/z, a retention time, and per-sample recovered intensities

## How to apply

First process each raw data file per batch with cdf.to.ftr (parameters: min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model), performing within-batch peak detection, quantification, RT correction and alignment so retention times are corrected within batch before any cross-batch comparison. Then collapse each batch's within-batch feature tables into a batch-level feature matrix (m/z plus mean RT and mean intensity via apLCMS feature.align with batch.tolerances). Align the batch-level matrices across batches using feature.align with the wider batch.align.mz.tol and batch.align.chr.tol tolerances. Finally run recover.weaker on every aligned batch-level feature against each file's original profile (recover.mz.range, recover.chr.range, use.observed.range, recover.min.count, intensity.weighted) to quantify features that were too weak for within-batch detection, assembling m/z, RT and per-sample recovered intensities into one cross-batch aligned feature table.

## Related tools

- **apLCMS** (Implements the entire two-stage batch-aware workflow: proc.cdf/cdf.to.ftr within-batch processing, feature.align within and across batches, and recover.weaker for weak signal recovery) — https://github.com/tianwei-yu/apLCMS
- **R** (Runtime environment for apLCMS; package dependencies (MASS, rgl, mzR, splines, doParallel, foreach) must be attached explicitly with library())
- **foreach** (Depends-level package supporting parallelized per-file/per-batch feature extraction in the apLCMS workflow)
- **doParallel** (Depends-level package providing the parallel backend (n.nodes) for foreach loops over raw files in cdf.to.ftr)
- **mzR** (Depends-level package for reading raw mass-spectrometry profile files in apLCMS)

## Examples

```
R: ftrs <- cdf.to.ftr(folder, file.pattern, n.nodes=4, min.exp=2, min.pres=0.5, min.run=12, mz.tol=1e-5)
```

## Evaluation signals

- aligned_batch_features.csv exists after the run.
- Every aligned row in the output carries an m/z, a retention time, and recovered intensities for the samples.
- No feature is lost between the batch-level alignment and the final recovered table.
- Feature detection and quantification consistency improves relative to single-group preprocessing (the paper's headline result: 'better consistency, as well as better down-stream analysis results').
- Cross-batch tolerances used for the second feature.align call are wider than the within-batch tolerances (batch.align.mz.tol/batch.align.chr.tol vs. mz.tol/chr.tol).

## Limitations

- Post-hoc batch effect correction methods can only adjust signal intensity; they cannot fix misalignment of features across batches or incomplete weak signal recovery — hence batching must be handled in preprocessing itself.
- No apLCMS package version is stated in the article or repository context.
- The article's discussion does not provide explicit limitations of the new method beyond the comparative claims.

## Evidence

- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [intro] we propose a new approach that preprocess the data in a two-stage manner: "To tackle the afore-mentioned problems, we propose a new approach that preprocess the data in a two-stage manner. The method directly uses the batch information to allow optimal within-batch and"
- [intro] such approaches can only adjust signal intensity: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery from the original data."
- [intro] including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery: "including peak detection, peak quantification, retention time (RT) correction, feature alignment, and weak signal recovery"
- [methods] cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model: "`cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
