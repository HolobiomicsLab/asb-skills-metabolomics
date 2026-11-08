---
name: spatial-metabolomics-feature-evaluation
description: Use when a trained machine-learning annotation or scoring model (e.g.,
  GBDT over ion features) exists and you need per-feature contribution/importance
  ratios to explain target-decoy separation, or to justify re-weighting or re-defining
  scoring features.
when_to_use_negative:
- No trained ML model is available and you only have raw imaging MS spectra — feature
  evaluation requires a fitted GBDT (or similar) model to explain
- You need FDR-controlled metabolite annotation output rather than model interpretability
  — use the METASPACE-ML annotation pipeline instead
- The model is not tree-based; shap.TreeExplainer is specific to tree ensembles such
  as GBDT and requires a different SHAP explainer for other model classes
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: METASPACE-ML
  role: Source of the trained GBDT ranking model and the five-feature representation
    (rho_spatial, rho_spectral, rho_chaos, absolute/relative m/z error) whose feature
    contributions are evaluated
  repo: https://github.com/metaspace2020/metaspace
- name: SHAP
  role: TreeExplainer-based Shapley value computation over the GBDT model to obtain
    per-feature mean absolute contributions
- name: METASPACE
  role: Knowledge base providing the public spatial metabolomics datasets used to
    train and test the model whose features are evaluated
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:8a4a27d557b22e3a80e3d0c63d4bebf9b9561e73234c2309710172bbcc0f931c
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-metabolomics-feature-evaluation

## Summary

Quantify how individual ion-scoring features (rho_spatial, rho_spectral, rho_chaos, absolute and relative m/z error) contribute to a trained METASPACE-ML gradient-boosted-tree ranking model using mean absolute SHAP values, revealing which features drive target/decoy discrimination in imaging mass spectrometry annotation.

## When to use

Use when a trained machine-learning annotation or scoring model (e.g., GBDT over ion features) exists and you need per-feature contribution/importance ratios to explain target-decoy separation, or to justify re-weighting or re-defining scoring features.

## When NOT to use

- No trained ML model is available and you only have raw imaging MS spectra — feature evaluation requires a fitted GBDT (or similar) model to explain
- You need FDR-controlled metabolite annotation output rather than model interpretability — use the METASPACE-ML annotation pipeline instead
- The model is not tree-based; shap.TreeExplainer is specific to tree ensembles such as GBDT and requires a different SHAP explainer for other model classes

## Inputs

- Trained METASPACE-ML gradient-boosted-tree (GBDT) ranking model
- Feature matrix of target/decoy candidate annotations with five features: rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error
- Target/decoy candidate annotations from testing datasets (e.g., 720 animal datasets)
- Reported per-feature contribution averages for validation (rho_spectral ~65%, rho_chaos ~5%)

## Outputs

- Per-feature normalized contribution-ratio table (shap_contribution_table.csv) summing to 100%
- Bar plot of feature contribution ratios (shap_contribution_plot.png)
- Ranked list of features by mean absolute SHAP value identifying dominant and minor contributors

## How to apply

Load the trained METASPACE-ML GBDT ranking model together with the feature matrix covering the five ion features (rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error) over target/decoy candidate annotations (e.g., the 720 animal testing datasets). Apply shap.TreeExplainer to compute SHAP values for all candidate annotations, then aggregate mean absolute SHAP values per feature and normalize to contribution ratios summing to 100%. Compare the recomputed ratios against reported averages — rho_spectral dominant (~65%) and rho_chaos minor (~5%) — and tabulate all five features. The rationale is that METASPACE's rule-based MSM scoring assigns equal weights to features, so SHAP attribution exposes which features actually drive the ML model's decisions and where feature re-definition is needed. Render a matplotlib bar plot of contribution ratios and persist both the table (shap_contribution_table.csv) and figure (shap_contribution_plot.png).

## Related tools

- **METASPACE-ML** (Source of the trained GBDT ranking model and the five-feature representation (rho_spatial, rho_spectral, rho_chaos, absolute/relative m/z error) whose feature contributions are evaluated) — https://github.com/metaspace2020/metaspace
- **SHAP** (TreeExplainer-based Shapley value computation over the GBDT model to obtain per-feature mean absolute contributions)
- **METASPACE** (Knowledge base providing the public spatial metabolomics datasets used to train and test the model whose features are evaluated) — https://github.com/metaspace2020/metaspace

## Examples

```
import shap; explainer = shap.TreeExplainer(gbdt_model); sv = explainer.shap_values(feature_matrix); ratios = abs(sv).mean(0); ratios = 100*ratios/ratios.sum()
```

## Evaluation signals

- Normalized per-feature SHAP contribution ratios sum to 100% across the five features
- rho_spectral emerges as the dominant feature with average contribution near the reported ~65%
- rho_chaos shows minor impact near the reported ~5%, consistent with target-decoy separation being mainly driven by rho_spectral
- Computed table (shap_contribution_table.csv) contains all five features with finite, non-negative mean absolute SHAP values
- Bar plot (shap_contribution_plot.png) renders with the five feature labels and matches the tabulated ratios

## Limitations

- rho_chaos6 had only a minor impact on model performance, so feature rankings can justify re-defining spatial informativeness quantification rather than confirming existing feature design
- Training data selection was biased towards public METASPACE datasets (from 47 labs), so contribution estimates may not generalize to underrepresented contexts such as QTOF-based datasets or single-cell metabolomics
- The two new m/z error features improved model accuracy only slightly, so their SHAP contributions should not be over-interpreted as decisive

## Evidence

- [discussion] Minor impact of rho_chaos on target-decoy separation: "rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral"
- [intro] Equal-weight limitation of rule-based MSM scoring motivating feature evaluation: "a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features"
- [discussion] GBDT robustness underpinning the model being explained: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [results] Reported contribution averages used for validation: [evidence span withheld — anchor status: not_found]
- [readme] Repository hosting the METASPACE engine and model code: "The METASPACE platform hosts an engine for
 metabolite annotation of imaging mass spectrometry data"
