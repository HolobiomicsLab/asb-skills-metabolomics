---
name: shap-feature-contribution-analysis
description: Use when a trained tree-based model (e.g. GBDT ranker) must be interpreted
  by ranking input features by average |SHAP| contribution — e.g. to identify which
  scores dominate target/decoy separation.
when_to_use_negative:
- The model is not tree-based (e.g. a linear model or neural network) — use shap.LinearExplainer
  or KernelExplainer-style attribution instead of TreeExplainer
- The analysis goal is model performance evaluation (e.g. FDR estimation quality)
  rather than feature-attribution explanation
- Input is already a feature-importance table — go directly to visualization rather
  than recomputing SHAP values
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: METASPACE-ML
  role: Source of the trained GBDT ranking model and its five-feature matrix over
    target/decoy candidate annotations
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Knowledge base from which the 720 animal testing datasets and candidate annotations
    are derived
  repo: https://github.com/metaspace2020/metaspace
- name: SHAP
  role: TreeExplainer-based computation of per-feature attribution values aggregated
    into contribution ratios
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:158f71be711f0c3ab88ea893a0877033b68094a2801652d7101b8afa6a44f552
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  - SHAP
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# shap-feature-contribution-analysis

## Summary

Quantify per-feature contributions of a gradient-boosted-tree (GBDT) model by aggregating mean absolute SHAP values into normalized contribution ratios, as used to explain how METASPACE-ML's five ion features (rho_spatial, rho_spectral, rho_chaos, absolute and relative m/z error) drive target/decoy discrimination.

## When to use

Use when a trained tree-based model (e.g. GBDT ranker) must be interpreted by ranking input features by average |SHAP| contribution — e.g. to identify which scores dominate target/decoy separation.

## When NOT to use

- The model is not tree-based (e.g. a linear model or neural network) — use shap.LinearExplainer or KernelExplainer-style attribution instead of TreeExplainer
- The analysis goal is model performance evaluation (e.g. FDR estimation quality) rather than feature-attribution explanation
- Input is already a feature-importance table — go directly to visualization rather than recomputing SHAP values

## Inputs

- Trained gradient-boosted-tree (GBDT) ranking model (METASPACE-ML pickle/model object)
- Feature matrix over target/decoy candidate annotations from the 720 animal testing datasets with five features: rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error
- Label vector distinguishing target vs decoy candidate annotations

## Outputs

- shap_contribution_table.csv — per-feature mean |SHAP| values and normalized contribution ratios summing to 100%
- shap_contribution_plot.png — matplotlib bar plot of feature contribution ratios
- Validation comparison against reported averages (rho_spectral 65%, rho_chaos 5%)

## How to apply

Load the trained GBDT model (e.g. the METASPACE-ML ranking model trained on 720 animal testing datasets) together with its feature matrix containing rho_spatial, rho_spectral, rho_chaos, absolute m/z error and relative m/z error over all target/decoy candidate annotations. Compute SHAP values over all candidates with shap.TreeExplainer, which exploits the tree structure for exact, efficient attribution. Aggregate mean absolute SHAP values per feature and normalize so per-feature ratios sum to 100%. Compare recomputed ratios against reference expectations (rho_spectral ~65%, rho_chaos ~5%) to validate that target-decoy separation is mainly driven by rho_spectral. Export a contribution table (e.g. shap_contribution_table.csv) and render a matplotlib bar plot (e.g. shap_contribution_plot.png).

## Related tools

- **METASPACE-ML** (Source of the trained GBDT ranking model and its five-feature matrix over target/decoy candidate annotations) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Knowledge base from which the 720 animal testing datasets and candidate annotations are derived) — https://github.com/metaspace2020/metaspace
- **SHAP** (TreeExplainer-based computation of per-feature attribution values aggregated into contribution ratios)

## Examples

```
import shap; explainer = shap.TreeExplainer(metaspace_ml_model); shap_vals = explainer.shap_values(X5); ratios = 100 * np.abs(shap_vals).mean(0) / np.abs(shap_vals).mean(0).sum()
```

## Evaluation signals

- Recomputed mean-|SHAP| contribution ratios sum to 100% across the five features
- rho_spectral emerges as the dominant feature at ~65% average contribution
- rho_chaos shows a minor contribution of ~5%, reproducing the reported model behavior
- All five features (rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error) appear in the tabulated output
- shap_contribution_table.csv and shap_contribution_plot.png are produced and their values agree

## Limitations

- rho_chaos6 had only a minor impact on model performance, so any feature-importance conclusion must acknowledge that target-decoy separation is mainly driven by rho_spectral and that spatial informativeness quantification may need re-definition
- Training data are biased towards public datasets represented in METASPACE (47 labs), so contribution ratios may not generalize to QTOF-based or single-cell metabolomics datasets that are underrepresented
- Decoy ions show heterogeneity in similarity to target ions, which can affect the interpretation of feature attributions for FDR estimation

## Evidence

- [discussion] rho_chaos had minor impact: "rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral"
- [abstract] METASPACE-ML is a machine learning-based approach: "METASPACE-ML is a machine learning-based approach addressing this challenge which incorporates new scores and computationally-efficient False Discovery Rate estimation"
- [discussion] GBDT robust to noisy outliers: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [discussion] two new m/z error features: "we proposed two new features estimating absolute and relative m/z error for centroided data, which helped improve the accuracy of the model predictions, albeit slightly"
- [readme] python-client in repo: [evidence span withheld — anchor status: external_doc]
