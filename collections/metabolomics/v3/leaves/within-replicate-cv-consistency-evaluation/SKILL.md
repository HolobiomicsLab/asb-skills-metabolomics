---
name: within-replicate-cv-consistency-evaluation
description: Use when comparing feature-detection/quantification consistency of preprocessing
  workflows (e.g. two-stage apLCMS vs traditional apLCMS vs XCMS) on replicated multi-batch
  LC/MS data with known replicate triplets.
when_to_use_negative:
- Data are not generated in multiple batches or lack replicate triplets, so within-triplet
  CV and batch-aware comparison are not meaningful
- Input is already a processed feature table with no access to the raw CDF profile
  files needed to re-run the preprocessing sweeps being compared
- The goal is differential-ablation or biological interpretation rather than evaluating
  preprocessing consistency across methods/thresholds
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: apLCMS
  role: R package providing cdf.to.ftr / proc.cdf / prof.to.features / feature.align
    and the two-stage batch-aware entry point whose aligned feature tables are evaluated
    for zero proportion and within-triplet CV
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Environment in which the metrics are computed (zero proportion, within-triplet
    CV) and the Fig. 3a/3c threshold plots are assembled; apLCMS dependencies include
    mzR, splines, doParallel, foreach, MASS, rgl
- name: doParallel
  role: Supplies parallel backend (with foreach) for the per-threshold cdf.to.ftr
    runs whose outputs are evaluated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:717dfc2792f7e743074cb1751f67ef4987c4eed6820bfea265d92cb28c17f033
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# within-replicate-cv-consistency-evaluation

## Summary

Evaluates consistency of LC/MS metabolomics preprocessing pipelines by computing the proportion of zeros in the aligned feature matrix and the within-replicate-triplet coefficient of variation (CV), as used to compare two-stage batch-aware apLCMS against traditional apLCMS and XCMS across min.exp threshold sweeps on the multi-batch ST000868 yeast dataset.

## When to use

Use when comparing feature-detection/quantification consistency of preprocessing workflows (e.g. two-stage apLCMS vs traditional apLCMS vs XCMS) on replicated multi-batch LC/MS data with known replicate triplets.

## When NOT to use

- Data are not generated in multiple batches or lack replicate triplets, so within-triplet CV and batch-aware comparison are not meaningful
- Input is already a processed feature table with no access to the raw CDF profile files needed to re-run the preprocessing sweeps being compared
- The goal is differential-ablation or biological interpretation rather than evaluating preprocessing consistency across methods/thresholds

## Inputs

- Final aligned feature-by-sample intensity matrix from each preprocessing workflow and threshold setting
- Batch/replicate-triplet annotation table (the 'info' input of two.stage.hybrid) identifying which samples are biological replicates within each batch
- Method and min.exp detection-threshold sweep configuration used to generate each feature matrix

## Outputs

- Proportion of zeros across all samples per aligned feature matrix
- Aggregated within-replicate-triplet CV per workflow and threshold
- Method-by-threshold consistency metrics table (e.g. fig3a_3c_consistency_metrics.csv) and plots of zero proportion and within-triplet CV versus min.exp threshold

## How to apply

Take each final aligned feature table produced by a preprocessing run (e.g. via cdf.to.ftr with a swept min.exp detection threshold on the ST000868 CDF profile files) and compute the proportion of zeros across all samples as a detection-consistency metric. Then, for each replicate triplet annotated in the batch/replicate 'info' table, compute the CV of each feature's intensities and aggregate the within-triplet CV as the quantification-consistency metric, adjusting for cell type and batch when comparing methods. Assemble method-by-threshold metrics into a single CSV (e.g. fig3a_3c_consistency_metrics.csv) and plot zero proportion and within-triplet CV versus min.exp threshold for each workflow. The rationale is that traditional single-group preprocessing can misalign peaks and incompletely recover weak signals, errors that post hoc batch correction cannot fix, so lower zeros and lower within-triplet CV after batch adjustment indicate the batch-aware two-stage pipeline (per-batch preprocessing plus cumulative nonlinear RT correction, cross-batch alignment, and weak-signal recovery controlled by recover.mz.range/recover.chr.range) is more consistent.

## Related tools

- **apLCMS** (R package providing cdf.to.ftr / proc.cdf / prof.to.features / feature.align and the two-stage batch-aware entry point whose aligned feature tables are evaluated for zero proportion and within-triplet CV) — https://github.com/tianwei-yu/apLCMS
- **R** (Environment in which the metrics are computed (zero proportion, within-triplet CV) and the Fig. 3a/3c threshold plots are assembled; apLCMS dependencies include mzR, splines, doParallel, foreach, MASS, rgl)
- **doParallel** (Supplies parallel backend (with foreach) for the per-threshold cdf.to.ftr runs whose outputs are evaluated)

## Examples

```
ftrs <- cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model)
```

## Evaluation signals

- Zero proportion is computable for every aligned feature matrix and is finite and in [0, 1] for each method-by-threshold combination
- Within-triplet CV is computed only over samples belonging to annotated replicate triplets, with cell type and batch accounted for in the comparison
- Lower within-cell-type-per-batch CVs for the two-stage method relative to traditional apLCMS and XCMS, as reported in the article, should be reproducible from the assembled metrics table
- Metrics table contains one row per method-by-min.exp-threshold combination matching the sweep design, and plotted curves (Fig. 3a zeros, Fig. 3c CV) vary smoothly with threshold

## Limitations

- Post hoc batch effect correction methods can only adjust signal intensity and cannot fix feature misalignment or incomplete weak-signal recovery, so this evaluation must compare preprocessing-stage outputs rather than corrected outputs
- The article context provides no explicit limitations of the new method and no apLCMS package version, so version-specific behavior cannot be pinned down
- Metric comparability depends on consistent batch/replicate-triplet annotation (the 'info' input) and identical threshold sweeps across methods

## Evidence

- [abstract] the new method resulted in feature tables with better consistency, as well as better down-stream analysis results: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [abstract] Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods: "Such practice can result in errors in the alignment of peaks, which cannot be corrected by post hoc application of batch effect correction methods"
- [discussion] we showed that the new method improved the consistency of feature detection and quantification: "we showed that the new method improved the consistency of feature detection and quantification"
- [methods] cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model: "cdf.to.ftr(folder, file.pattern, n.nodes, min.exp, min.pres, min.run, mz.tol, baseline.correct.noise.percentile, shape.model"
