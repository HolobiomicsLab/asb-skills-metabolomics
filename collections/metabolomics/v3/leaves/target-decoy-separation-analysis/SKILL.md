---
name: target-decoy-separation-analysis
description: Use when a trained GBDT ranking model over target/decoy metabolite annotations
  needs its per-feature contributions explained, e.g. to check whether rho_spectral
  dominates separation and rho_chaos is minor.
when_to_use_negative:
- You need FDR-controlled metabolite annotation of new imaging MS data rather than
  post-hoc model interpretation of feature importance
- The model is not a tree ensemble (SHAP TreeExplainer is specific to tree-based models
  like the GBDT ranking model)
- The feature matrix lacks the rho_chaos or m/z error features, making the five-feature
  contribution comparison impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: METASPACE-ML
  role: Source of the trained GBDT ranking model and the five target/decoy discrimination
    features being explained
  repo: https://github.com/metaspace2020/metaspace
- name: SHAP
  role: TreeExplainer computes per-feature SHAP values whose mean absolute magnitudes
    are normalized to contribution percentages
- name: METASPACE
  role: Platform and knowledge base providing the public imaging MS datasets underlying
    the model and test data
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:c2c4e2734a18a31e73955e4c13a691658e0214ceeeb0971663a0d8a4f63a3b2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  - SHAP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# target-decoy-separation-analysis

## Summary

Quantify which of the five METASPACE-ML ion features (rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error) drive target/decoy discrimination by computing per-feature mean absolute SHAP contributions from the gradient-boosted-tree ranking model.

## When to use

Use when a trained GBDT ranking model over target/decoy metabolite annotations needs its per-feature contributions explained, e.g. to check whether rho_spectral dominates separation and rho_chaos is minor.

## When NOT to use

- You need FDR-controlled metabolite annotation of new imaging MS data rather than post-hoc model interpretation of feature importance
- The model is not a tree ensemble (SHAP TreeExplainer is specific to tree-based models like the GBDT ranking model)
- The feature matrix lacks the rho_chaos or m/z error features, making the five-feature contribution comparison impossible

## Inputs

- Trained METASPACE-ML GBDT ranking model
- Feature matrix for 720 animal testing datasets with five features: rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error
- Target/decoy candidate annotations for each dataset

## Outputs

- Per-feature contribution table (shap_contribution_table.csv) with normalized mean absolute SHAP ratios summing to 100%
- Bar plot of contribution ratios (shap_contribution_plot.png)
- Comparison of recomputed ratios against reported values (rho_spectral ~65%, rho_chaos ~5%)

## How to apply

Load the trained METASPACE-ML gradient-boosted-tree (GBDT) ranking model together with the feature matrix for the 720 animal testing datasets containing the five features (rho_spatial, rho_spectral, rho_chaos, absolute m/z error, relative m/z error). Run TreeExplainer from SHAP over all target/decoy candidate annotations, aggregate mean absolute SHAP values per feature, and normalize to per-feature contribution ratios summing to 100%. Compare against the reported averages (rho_spectral ~65%, rho_chaos ~5%) and tabulate all five features, then render a matplotlib bar plot saved alongside the table (shap_contribution_table.csv, shap_contribution_plot.png). GBDT is chosen because it is known to be robust to noisy outliers that can distort target/decoy separation.

## Related tools

- **METASPACE-ML** (Source of the trained GBDT ranking model and the five target/decoy discrimination features being explained) — https://github.com/metaspace2020/metaspace
- **SHAP** (TreeExplainer computes per-feature SHAP values whose mean absolute magnitudes are normalized to contribution percentages)
- **METASPACE** (Platform and knowledge base providing the public imaging MS datasets underlying the model and test data) — https://github.com/metaspace2020/metaspace

## Examples

```
import shap; shap_values = shap.TreeExplainer(gbdt_model).shap_values(X); contrib = shap_values.abs().mean(0); (contrib/contrib.sum()*100).to_csv('shap_contribution_table.csv')
```

## Evaluation signals

- Recomputed contribution ratios normalize to 100% across the five features
- rho_spectral emerges as the dominant contributor at approximately 65% average contribution
- rho_chaos is the lowest-impact feature at approximately 5%
- Aggregated mean absolute SHAP values are computed over all target/decoy candidates in the 720 animal datasets, not a subsample
- Both shap_contribution_table.csv and shap_contribution_plot.png are produced and consistent with each other

## Limitations

- Target-decoy separation being driven mainly by rho_spectral demands re-defining the quantification of spatial informativeness
- rho_chaos had only a minor impact on model performance, so it contributes little discriminating signal
- Decoy ions show heterogeneity in similarity to target ions, and the most reliable decoy generation method needs further investigation

## Evidence

- [discussion] rho_chaos minor impact and rho_spectral dominance: "rho_chaos6 had only a minor impact on the model performance (Fig. 6A), and target-decoy separation was mainly driven by rho_spectral"
- [discussion] GBDT robustness rationale: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [discussion] new m/z error features: "we proposed two new features estimating absolute and relative m/z error for centroided data, which helped improve the accuracy"
- [readme] METASPACE platform scope: "metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets"
