---
name: mz-error-feature-contribution-quantification
description: Use when a trained METASPACE-ML GBDT ranking model and its candidate-annotation
  feature matrix are available and the goal is to attribute the prediction score back
  to individual ion features, especially to check whether rho_spectral or the m/z
  error features dominate target-decoy separation.
when_to_use_negative:
- The model to explain is not a tree ensemble (e.g., a linear or kernel model), since
  shap.TreeExplainer is specific to tree-based models like the METASPACE-ML GBDT
- The feature matrix does not contain the five METASPACE-ML ion features (rho_spatial,
  rho_spectral, rho_chaos, absolute/relative m/z error), so contribution ratios would
  not map to the published breakdown
- The goal is to assess annotation accuracy or FDR performance against rule-based
  MSM rather than to explain which features drive the prediction score
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: METASPACE-ML
  role: Provides the trained GBDT ranking model and the five-feature representation
    (including the absolute and relative m/z error features) whose predictions are
    decomposed via SHAP
  repo: https://github.com/metaspace2020/metaspace
- name: SHAP
  role: TreeExplainer-based additive attribution used to compute per-candidate SHAP
    values that are averaged and normalized to feature contribution ratios
- name: METASPACE
  role: Annotation engine and knowledge base whose public imaging MS datasets underpin
    the trained model and the candidate annotations analyzed
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:a76eec802ec34e0e385e7639e01b722da4e86156ebcd6659860aae7e79f6697a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  - SHAP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mz-error-feature-contribution-quantification

## Summary

Quantifies how the five METASPACE-ML features (rho_spatial, rho_spectral, rho_chaos, absolute and relative m/z error) differentially drive target-decoy discrimination by computing SHAP values on the trained gradient-boosted-tree ranking model over 720 animal testing datasets and normalizing mean absolute SHAP values to per-feature contribution percentages.

## When to use

Use when a trained METASPACE-ML GBDT ranking model and its candidate-annotation feature matrix are available and the goal is to attribute the prediction score back to individual ion features, especially to check whether rho_spectral or the m/z error features dominate target-decoy separation.

## When NOT to use

- The model to explain is not a tree ensemble (e.g., a linear or kernel model), since shap.TreeExplainer is specific to tree-based models like the METASPACE-ML GBDT
- The feature matrix does not contain the five METASPACE-ML ion features (rho_spatial, rho_spectral, rho_chaos, absolute/relative m/z error), so contribution ratios would not map to the published breakdown
- The goal is to assess annotation accuracy or FDR performance against rule-based MSM rather than to explain which features drive the prediction score

## Inputs

- Trained METASPACE-ML gradient-boosted-tree (GBDT) ranking model
- Feature matrix of target/decoy candidate annotations for the 720 animal testing datasets with the five features rho_spatial, rho_spectral, rho_chaos, absolute m/z error, and relative m/z error

## Outputs

- Per-feature normalized mean absolute SHAP contribution ratios summing to 100% (e.g., shap_contribution_table.csv)
- Bar plot of feature contribution ratios (e.g., shap_contribution_plot.png)

## How to apply

Load the trained METASPACE-ML gradient-boosted-tree ranking model together with the feature matrix for the 720 animal testing datasets containing the five features (rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error). Run SHAP via shap.TreeExplainer over all target and decoy candidate annotations to obtain per-candidate additive attributions. Aggregate mean absolute SHAP values per feature and normalize them so the five contribution ratios sum to 100%. Validate the result against the reported averages: rho_spectral should dominate (~65%) and rho_chaos should be minor (~5%), reflecting that target-decoy separation is driven mainly by rho_spectral. Finally tabulate all five ratios (e.g., shap_contribution_table.csv) and render a matplotlib bar plot (e.g., shap_contribution_plot.png).

## Related tools

- **METASPACE-ML** (Provides the trained GBDT ranking model and the five-feature representation (including the absolute and relative m/z error features) whose predictions are decomposed via SHAP) — https://github.com/metaspace2020/metaspace
- **SHAP** (TreeExplainer-based additive attribution used to compute per-candidate SHAP values that are averaged and normalized to feature contribution ratios)
- **METASPACE** (Annotation engine and knowledge base whose public imaging MS datasets underpin the trained model and the candidate annotations analyzed) — https://github.com/metaspace2020/metaspace

## Examples

```
import shap; sv = shap.TreeExplainer(model).shap_values(X); contrib = abs(sv).mean(0); ratio = contrib / contrib.sum() * 100
```

## Evaluation signals

- Normalized mean absolute SHAP ratios across the five features sum to exactly 100%
- rho_spectral contribution is dominant at approximately 65%, matching the reported average
- rho_chaos contribution is minor at approximately 5%, matching the reported average
- All five features (rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error) appear in the tabulated output with a finite contribution ratio
- Recomputed ratios reproduce the reported ordering of features (rho_spectral > ... > rho_chaos)

## Limitations

- rho_chaos had only a minor impact on model performance, so contributions attributed to it are small and the quantification mainly reflects rho_spectral-driven separation
- The two m/z error features for centroided data improved model prediction accuracy only slightly, so their SHAP contributions are expected to be modest
- The training/testing data selection is biased towards public datasets represented in METASPACE (from 47 labs), so contribution estimates may not generalize to QTOF or single-cell/cultured-cell datasets that are underrepresented

## Evidence

- [discussion] rho_chaos minor impact; target-decoy separation driven mainly by rho_spectral: "rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral"
- [discussion] m/z error features improved predictions slightly: "we proposed two new features estimating absolute and relative m/z error for centroided data, which helped improve the accuracy of the model predictions"
- [discussion] GBDT robustness to noisy outliers: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [other] SHAP feature-importance workflow on 720 animal datasets: [evidence span withheld — anchor status: not_found]
- [readme] Python client and notebooks for annotations database analysis: [evidence span withheld — anchor status: external_doc]
