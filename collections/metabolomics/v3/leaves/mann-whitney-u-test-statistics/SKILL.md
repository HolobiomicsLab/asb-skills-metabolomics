---
name: mann-whitney-u-test-statistics
description: Reference for testing whether median feature intensities differ across
  outcome categories (correct vs incorrect vs no annotation) when you need a nonparametric,
  rank-based significance test rather than a parametric t-test.
when_to_use_negative:
- When the comparison groups are derived from the same feature measured under paired
  conditions — use a paired Wilcoxon signed-rank or paired t-test instead
- When feature intensities are known to be approximately normal and equal-variance
  across groups, a t-test yields more power at the same sample size
- When the goal is to compare annotation success rates (proportions) rather than intensity
  distributions — use a chi-squared or Fisher's exact test
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: R package whose annotation output (rank, intensity, dataset) supplies the
    per-feature labels and intensities the Mann–Whitney test is run on
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Statistical environment in which the Mann–Whitney U tests are performed
  repo: https://github.com/gggraca/MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:f4b87e79b37ae068332e79d212140eb5167422b98ab6e40622814fe149d51e08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mann-whitney-u-test-statistics

## Summary

Mann–Whitney U statistics are used to determine whether feature intensity distributions differ significantly across annotation outcome classes, with the p<0.05 significance threshold indicating where MetaboAnnotatoR's rank-1 annotations are intensity-dependent rather than class-independent.

## When to use

Reference for testing whether median feature intensities differ across outcome categories (correct vs incorrect vs no annotation) when you need a nonparametric, rank-based significance test rather than a parametric t-test.

## When NOT to use

- When the comparison groups are derived from the same feature measured under paired conditions — use a paired Wilcoxon signed-rank or paired t-test instead
- When feature intensities are known to be approximately normal and equal-variance across groups, a t-test yields more power at the same sample size
- When the goal is to compare annotation success rates (proportions) rather than intensity distributions — use a chi-squared or Fisher's exact test

## Inputs

- Annotation output table with per-feature intensity and annotation rank/outcome per dataset (Lipid+, Lipid−, HILIC+ serum datasets)
- Expert manual annotation labels marking each feature as rank 1, incorrect, or non-annotated

## Outputs

- Mann–Whitney U statistic and p-value per dataset for each class pair (rank 1 vs non-annotated, rank 1 vs incorrect)
- Median feature intensity per annotation class per dataset
- Significance call (p < 0.05) indicating intensity-dependence of annotation success

## How to apply

Group annotated LC–MS features into three outcome classes (rank 1 correct, incorrectly annotated, or non-annotated) using expert manual annotations as ground truth, then extract each feature's peak intensity and compute the median per class within each dataset (Lipid+, Lipid−, HILIC+). Run a Mann–Whitney U test on the intensity distributions between class pairs — rank 1 vs non-annotated, and rank 1 vs incorrectly annotated — within each of the three datasets, marking significance at p < 0.05. This test is appropriate because feature intensities across annotation classes are not assumed to be normally distributed, so the rank-based statistic avoids parametric distributional assumptions while still yielding a signed direction (higher median intensity for rank 1 hits in the Lipid− and HILIC+ datasets). The test is a post-hoc diagnostic applied after the annotation workflow, not part of the annotation algorithm itself.

## Related tools

- **MetaboAnnotatoR** (R package whose annotation output (rank, intensity, dataset) supplies the per-feature labels and intensities the Mann–Whitney test is run on) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Statistical environment in which the Mann–Whitney U tests are performed) — https://github.com/gggraca/MetaboAnnotatoR

## Examples

```
R> wilcox.test(intensity ~ annotation_class, data = lipid_minus_features, exact = FALSE)
```

## Evaluation signals

- Mann–Whitney p-values for rank 1 vs non-annotated must be significant (p < 0.05) in Lipid− and HILIC+ but not in Lipid+, matching the paper's reported intensity-dependence pattern
- Mann–Whitney p-value for rank 1 vs incorrectly annotated must be significant (p < 0.05) in the Lipid− dataset only
- Median intensity for rank 1 annotations must be higher than non-annotated in Lipid− and HILIC+ and higher than incorrect annotations in Lipid−
- Sample sizes per class pair must be sufficient for the U statistic to be computed without ties-driven degeneracy
- The significance pattern must be reproducible when re-run with an independent Mann–Whitney implementation

## Limitations

- The test is nonparametric and rank-based, so it does not estimate effect size or direction of intensity difference beyond the sign of the median comparison; report medians alongside p-values
- Only class-pair contrasts supported by the study design (rank 1 vs non-annotated, rank 1 vs incorrect) were tested; other contrasts (e.g. incorrect vs non-annotated) were not evaluated
- Significance at p < 0.05 is assessed per dataset without multiple-testing correction across the three datasets × two contrasts, so isolated significant results should be interpreted as descriptive intensity trends rather than confirmed effects
- Intensity distributions across annotation classes can differ by dataset chemistry (lipid-dominated vs HILIC), so results are not transferable across datasets

## Evidence

- [results] Median feature intensities differ significantly across annotation outcome classes: [evidence span withheld — anchor status: not_found]
- [abstract] The three human serum datasets supply the feature-intensity and annotation-rank table the test operates on: "We demonstrate the workflow in three human serum datasets containing 599 features manually annotated by experts."
- [abstract] The test is applied as a post-hoc diagnostic on MetaboAnnotatoR output: "The workflow is freely available as an open-source R package, MetaboAnnotatoR, together with the fragment libraries from Github"
- [intro] Class labels come from expert manual annotation used as ground truth: "Accurate metabolite annotation is a largely manual and time-consuming process that typically consists of MS spectral inspection and additional MS/MS-targeted fragmentation experiments"
- [abstract] The paper's own evaluation metric for annotation success is precision and recall, which the intensity-based test complements: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)."
