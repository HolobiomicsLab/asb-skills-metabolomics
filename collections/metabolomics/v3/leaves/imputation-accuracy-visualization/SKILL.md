---
name: imputation-accuracy-visualization
description: Use when you need to visually verify, for a single dataset with mixed
  MNAR (below-LOD) and MAR missingness, how truncation-based KNN imputation compares
  against Euclidean or correlation-based KNN imputation on a subset of metabolites.
when_to_use_negative:
- When the goal is aggregate quantitative comparison across many simulations — use
  RMSE or the metabolite list concordance index (MLCI) instead of a single-dataset
  visual inspection
- When the dataset has no below-detection-limit (MNAR) missingness, since the plot's
  key comparison region is the shaded below-LOD area
- When metabolite distributions are strongly non-normal, since KNN-TN's truncated-normal
  estimation assumes normality and its imputations may not show the expected advantage
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: KNN-TN
  role: Truncation-based KNN imputation whose imputed values are plotted as blue triangles
    for comparison below the LOD
- name: KNN-CR
  role: Correlation-distance KNN imputation whose imputed values are plotted as red
    squares
- name: KNN-EU
  role: Euclidean-distance KNN imputation whose imputed values are plotted as green
    diamonds
- name: Newton-Raphson optimization for truncated normal MLE
  role: Estimates the truncated-normal mean and SD used inside KNN-TN before its imputed
    values are visualized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:3319cde25e180796b66ae084cac5094c53d43446afce154562caeaacf6df6d51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KNN-CR
  - KNN-CR (correlation-based KNN imputation)
  - KNN-EU
  - KNN-EU (Euclidean-based KNN imputation)
  - KNN-TN
  - KNN-TN (truncation-based KNN imputation)
  - Newton-Raphson optimization for truncated normal MLE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# imputation-accuracy-visualization

## Summary

Produces a per-metabolite scatter plot (as in the paper's Figure 6) that overlays observed values, true missing values, and imputed values from KNN-TN, KNN-CR, and KNN-EU for a single simulated metabolomics dataset, with the below-LOD region shaded so MNAR imputation accuracy can be visually inspected.

## When to use

Use when you need to visually verify, for a single dataset with mixed MNAR (below-LOD) and MAR missingness, how truncation-based KNN imputation compares against Euclidean or correlation-based KNN imputation on a subset of metabolites.

## When NOT to use

- When the goal is aggregate quantitative comparison across many simulations — use RMSE or the metabolite list concordance index (MLCI) instead of a single-dataset visual inspection
- When the dataset has no below-detection-limit (MNAR) missingness, since the plot's key comparison region is the shaded below-LOD area
- When metabolite distributions are strongly non-normal, since KNN-TN's truncated-normal estimation assumes normality and its imputations may not show the expected advantage

## Inputs

- Single simulated metabolomics matrix (e.g., N=50 samples x M=400 metabolites) drawn from a multivariate normal distribution
- Missingness mask imposing 15% missing values (10% MNAR below the LOD, 5% MAR)
- Limit of detection (LOD) threshold defining the truncation point
- Three sets of imputed values from KNN-TN, KNN-CR, and KNN-EU (K=10 neighbors)
- True (pre-masking) values for the missing entries

## Outputs

- Per-metabolite comparison figure (figure6_imputation_comparison.png) with open black circles (observed), black stars (true missing), blue triangles (KNN-TN), red squares (KNN-CR), green diamonds (KNN-EU), and a light-red shaded below-LOD region

## How to apply

Generate one simulated dataset (e.g., 50 samples x 400 metabolites) from a multivariate normal distribution and impose 15% missingness split as 10% MNAR below the LOD and 5% MAR, log-transforming metabolites first to approximately achieve normality prior to imputation. Impute the missing entries three times with KNN using K=10 neighbors: KNN-TN (truncation-based, with truncated-normal mean and SD estimated by Newton-Raphson MLE), KNN-CR (correlation-distance metric), and KNN-EU (Euclidean-distance metric). Restrict the plot to a small panel of metabolites (the paper uses the first 20) and, for each, plot observed values as open black circles, true missing values as black stars, and imputed values as KNN-TN blue triangles, KNN-CR red squares, and KNN-EU green diamonds. Shade the region below the limit of detection in light red so that MNAR imputations can be inspected against the LOD. The expected pattern is that KNN-TN places below-LOD imputations closer to the true (star) values than KNN-CR or KNN-EU.

## Related tools

- **KNN-TN** (Truncation-based KNN imputation whose imputed values are plotted as blue triangles for comparison below the LOD)
- **KNN-CR** (Correlation-distance KNN imputation whose imputed values are plotted as red squares)
- **KNN-EU** (Euclidean-distance KNN imputation whose imputed values are plotted as green diamonds)
- **Newton-Raphson optimization for truncated normal MLE** (Estimates the truncated-normal mean and SD used inside KNN-TN before its imputed values are visualized)

## Examples

```
ax.plot(x, obs, 'ko', mfc='none'); ax.plot(x, true, 'k*'); ax.plot(x, tn, 'b^'); ax.plot(x, cr, 'rs'); ax.plot(x, eu, 'gd'); ax.axhspan(ymin_lod, ax_top, color='mistyrose'); plt.savefig('figure6_imputation_comparison.png')
```

## Evaluation signals

- Figure renders the expected marker grammar: open black circles for observed, black stars for true missing, blue triangles (KNN-TN), red squares (KNN-CR), green diamonds (KNN-EU)
- The region below the LOD is shaded light red and below-LOD imputed points are visually distinguishable from the LOD boundary
- For metabolites where missing values fall below the LOD (e.g., metabolites 1, 3, 4, 7, 8, 12, 13 in the paper's Figure 6), KNN-TN triangles lie closer to the true stars than KNN-CR squares or KNN-EU diamonds
- Exactly the first 20 metabolites are shown, matching the paper's single-dataset comparison scope
- Output file is saved under the expected name (figure6_imputation_comparison.png)

## Limitations

- Visual inspection is qualitative; it complements but does not replace RMSE or MCI-based quantitative evaluation of imputation accuracy
- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, so the visual advantage may not hold for non-normal metabolite distributions
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, which can make KNN-TN imputed points appear erratic in the plot

## Evidence

- [results] KNN-TN below-LOD advantage in Fig. 6: [evidence span withheld — anchor status: not_found]
- [abstract] KNN-TN improved performance under MAR + LOD: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
- [discussion] Log transform before imputation: "we log transform them to approximately achieve normality prior to imputation"
- [discussion] Truncated normal instability caveat: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution, which"
