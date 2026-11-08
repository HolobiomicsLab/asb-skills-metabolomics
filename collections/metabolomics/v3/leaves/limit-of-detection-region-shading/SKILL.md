---
name: limit-of-detection-region-shading
description: Use when reproducing single-dataset imputation comparisons (e.g., Figure
  6-style plots) where missing values are partly MNAR below a detection limit and
  below-LOD imputation accuracy must be inspected visually.
when_to_use_negative:
- The dataset has purely MAR/MCAR missingness with no detection limit, so there is
  no below-LOD region to shade and no MNAR regime to inspect
- The goal is a quantitative multi-dataset comparison (RMSE or MLCI tables) rather
  than visual inspection of imputations for a single dataset
- Missing values are substituted by simple constant imputation (minimum value, zero,
  metabolite mean) where no per-entry imputed values exist to overlay against the
  LOD
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: KNN-TN
  role: Produces the truncation-based imputed values (truncated-normal mean and SD
    via Newton-Raphson MLE) plotted as blue triangles against the shaded LOD region
- name: KNN-CR
  role: Produces correlation-distance KNN imputations plotted as red squares for visual
    comparison below the LOD
- name: KNN-EU
  role: Produces Euclidean-distance KNN imputations plotted as green diamonds for
    visual comparison below the LOD
- name: matplotlib
  role: Renders the scatter overlay and the light-red axhspan/axvspan shading below
    the LOD, saving figure6_imputation_comparison.png
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:9b662647d98be2a388418dfcd0bd21524339c3f35994f694d7a54d7acfed827a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KNN-CR
  - KNN-EU
  - KNN-TN
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# limit-of-detection-region-shading

## Summary

A visualization skill from Shah et al. (2017, BMC Bioinformatics) that shades the below-limit-of-detection region in light red on an imputation-comparison plot, so that MNAR imputed values from KNN-TN, KNN-CR, and KNN-EU can be visually inspected against the LOD threshold.

## When to use

Use when reproducing single-dataset imputation comparisons (e.g., Figure 6-style plots) where missing values are partly MNAR below a detection limit and below-LOD imputation accuracy must be inspected visually.

## When NOT to use

- The dataset has purely MAR/MCAR missingness with no detection limit, so there is no below-LOD region to shade and no MNAR regime to inspect
- The goal is a quantitative multi-dataset comparison (RMSE or MLCI tables) rather than visual inspection of imputations for a single dataset
- Missing values are substituted by simple constant imputation (minimum value, zero, metabolite mean) where no per-entry imputed values exist to overlay against the LOD

## Inputs

- Simulated metabolomics matrix (N x M, e.g., 50 x 400) with imposed 15% missingness (10% MNAR below LOD, 5% MAR)
- True (pre-masking) values for the missing entries
- Imputed matrices from KNN-TN, KNN-CR, and KNN-EU (K = 10 neighbors each)
- Scalar limit-of-detection threshold used as the truncation point
- Observed (non-missing) values for overlay

## Outputs

- figure6_imputation_comparison.png — per-metabolite scatter with open black circles (observed), black stars (true missing), blue triangles (KNN-TN), red squares (KNN-CR), green diamonds (KNN-EU), and a light-red shaded below-LOD region

## How to apply

Generate one simulated dataset (e.g., N = 50 samples by M = 400 metabolites from a multivariate normal) with 15% missingness split as 10% MNAR below the LOD and 5% MAR, then impute it three times with KNN-TN (truncated-normal mean/SD via Newton-Raphson MLE), KNN-CR (correlation distance), and KNN-EU (Euclidean distance), each with K = 10 neighbors. Restrict the plot to the first 20 metabolites, plotting observed values as open black circles, true missing values as black stars, and imputed values as KNN-TN blue triangles, KNN-CR red squares, and KNN-EU green diamonds. Shade the region below the limit of detection in light red so MNAR imputations fall visibly inside the shaded band; this is the decision point at which a reviewer can judge whether KNN-TN recovers below-LOD values better than the distance-metric variants. Save the result as figure6_imputation_comparison.png. The rationale is that the paper's key advantage claim for KNN-TN concerns values below the detection limit, so without the shaded LOD band the MNAR regime cannot be distinguished from MAR imputations on the figure.

## Related tools

- **KNN-TN** (Produces the truncation-based imputed values (truncated-normal mean and SD via Newton-Raphson MLE) plotted as blue triangles against the shaded LOD region)
- **KNN-CR** (Produces correlation-distance KNN imputations plotted as red squares for visual comparison below the LOD)
- **KNN-EU** (Produces Euclidean-distance KNN imputations plotted as green diamonds for visual comparison below the LOD)
- **matplotlib** (Renders the scatter overlay and the light-red axhspan/axvspan shading below the LOD, saving figure6_imputation_comparison.png)

## Examples

```
import matplotlib.pyplot as plt; ax.axhspan(0, lod, color='lightcoral', alpha=0.3); plt.savefig('figure6_imputation_comparison.png')
```

## Evaluation signals

- The shaded light-red region boundary coincides exactly with the LOD threshold used as the truncation point for KNN-TN
- Marker styles match the specified scheme: open black circles (observed), black stars (true missing), blue triangles (KNN-TN), red squares (KNN-CR), green diamonds (KNN-EU)
- Only the first 20 metabolites of the dataset are shown, matching the paper's Figure 6 scope
- For metabolites where the paper reports KNN-TN advantage (e.g., metabolites 1, 3, 4, 7, 8, 12, 13), blue triangles falling in the shaded below-LOD band lie closer to black stars than red squares or green diamonds
- Output file is saved as figure6_imputation_comparison.png with labeled axes and an expert-review pass on shading, markers, and axis labels

## Limitations

- The figure is a single simulated dataset (N = 50, M = 400); it is illustrative, not a quantitative performance summary across datasets
- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, which metabolite data may violate, affecting what the shaded region reveals
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability that the visual comparison cannot fully convey

## Evidence

- [other] Shade the region below the limit of detection in light red so that MNAR imputations can be visually inspected against the LOD: [evidence span withheld — anchor status: not_found]
- [results] KNN-TN was in most cases able to impute missing values below the LOD better than KNN-CR and KNN-EU: [evidence span withheld — anchor status: not_found]
- [intro] missing values occurs either as MNAR (metabolites occur at low abundances, below the detection limit) or MAR: "In metabolomics studies, we assume that the missing values occurs either as MNAR (metabolites occur at low abundances, below the detection limit) or MAR, e.g., metabolites are truly not present or"
- [abstract] KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
