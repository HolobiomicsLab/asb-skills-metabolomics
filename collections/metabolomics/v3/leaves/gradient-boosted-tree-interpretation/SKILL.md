---
name: gradient-boosted-tree-interpretation
description: Use when you need to explain which input features drive a trained gradient-boosted-tree
  (GBDT) model's predictions, e.g. attributing METASPACE-ML's target/decoy prediction
  score to its five ion scores.
when_to_use_negative:
- The model is not a tree ensemble (TreeExplainer/SHAP tree attribution is specific
  to GBDT-style models)
- The goal is to evaluate annotation performance (precision, MAP scores) of METASPACE-ML
  rather than to explain feature contributions to its prediction score
- Only the rule-based MSM scoring of METASPACE is being analyzed — MSM assigns equal
  weights to features, so SHAP attribution does not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3810
- http://edamontology.org/topic_0092
tools:
- name: METASPACE-ML
  role: Source of the trained GBDT ranking model and its five-feature target/decoy
    prediction score being interpreted
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform hosting the annotation engine and knowledge base from which datasets
    and candidate annotations are drawn
  repo: https://github.com/metaspace2020/metaspace
- name: SHAP
  role: TreeExplainer-based computation of Shapley Additive explanation values for
    the GBDT model over all target/decoy candidates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:5e93ec30b13f21b157a68133f7cec467ba697f1dab4a610985f7563af361ec1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  - SHAP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3810
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gradient-boosted-tree-interpretation

## Summary

Reproduces the SHAP-based feature-importance analysis of METASPACE-ML's gradient-boosted-tree ranking model, quantifying how the five ion features (rho_spatial, rho_spectral, rho_chaos, absolute and relative m/z error) contribute to target-decoy discrimination in imaging mass spectrometry annotation.

## When to use

Use when you need to explain which input features drive a trained gradient-boosted-tree (GBDT) model's predictions, e.g. attributing METASPACE-ML's target/decoy prediction score to its five ion scores.

## When NOT to use

- The model is not a tree ensemble (TreeExplainer/SHAP tree attribution is specific to GBDT-style models)
- The goal is to evaluate annotation performance (precision, MAP scores) of METASPACE-ML rather than to explain feature contributions to its prediction score
- Only the rule-based MSM scoring of METASPACE is being analyzed — MSM assigns equal weights to features, so SHAP attribution does not apply

## Inputs

- Trained METASPACE-ML gradient-boosted-tree (GBDT) ranking model
- Feature matrix for 720 animal testing datasets with five features per target/decoy candidate annotation: rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error

## Outputs

- Per-feature mean absolute SHAP contribution ratios normalized to 100% (shap_contribution_table.csv)
- Bar plot of feature contribution ratios (shap_contribution_plot.png)

## How to apply

Load the trained METASPACE-ML GBDT ranking model together with the feature matrix for the 720 animal testing datasets, whose candidate annotations carry the five features rho_spatial, rho_spectral, rho_chaos, absolute m/z error, and relative m/z error. Compute SHAP values over all target/decoy candidate annotations using SHAP's TreeExplainer, which applies to tree ensembles like GBDT. Aggregate mean absolute SHAP values per feature and normalize them to contribution ratios summing to 100%. Compare the recomputed ratios against the reported averages (rho_spectral 65%, rho_chaos 5%) and render a bar plot of contribution ratios with matplotlib. The rationale is that GBDT feature attributions reveal which scores actually separate target from decoy ions, which matters because the article found rho_chaos had only a minor impact while rho_spectral drove target-decoy separation.

## Related tools

- **METASPACE-ML** (Source of the trained GBDT ranking model and its five-feature target/decoy prediction score being interpreted) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform hosting the annotation engine and knowledge base from which datasets and candidate annotations are drawn) — https://github.com/metaspace2020/metaspace
- **SHAP** (TreeExplainer-based computation of Shapley Additive explanation values for the GBDT model over all target/decoy candidates)

## Examples

```
import shap; explainer = shap.TreeExplainer(gbdt_ranking_model); vals = explainer.shap_values(X_720); contrib = abs(vals).mean(axis=0) / abs(vals).mean(axis=0).sum()
```

## Evaluation signals

- Recomputed average SHAP contributions reproduce the reported values: rho_spectral ≈ 65% (largest) and rho_chaos ≈ 5% (smallest)
- Normalized per-feature contribution ratios sum to 100%
- The table contains exactly five features: rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error
- shap_contribution_table.csv and shap_contribution_plot.png are both produced and agree on per-feature values
- Result is consistent with the paper's conclusion that target-decoy separation is mainly driven by rho_spectral, not rho_chaos

## Limitations

- rho_chaos had only a minor impact on model performance, and target-decoy separation was mainly driven by rho_spectral, which demands re-defining the quantification of spatial informativeness
- The two new m/z error features (absolute and relative) only improved model prediction accuracy slightly, so their SHAP contributions are expected to be modest
- Training/testing datasets are biased towards public METASPACE datasets (from 47 labs), and QTOF-based datasets are underrepresented, so SHAP attributions may not generalize to those contexts

## Evidence

- [discussion] rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral: "rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral"
- [discussion] GBDT robustness to outliers: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [discussion] two new m/z error features: "we proposed two new features estimating absolute and relative m/z error for centroided data, which helped improve the accuracy of the model"
- [readme] METASPACE platform description: "The METASPACE platform hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase"
