---
name: cross-validated-auroc-benchmarking
description: Use when comparing supervised multi-omics classifiers (molecular-level
  vs pathway-level) on semi-synthetic or real labelled data and you need unbiased
  generalisation performance reported as AUROC across effect sizes.
when_to_use_negative:
- When the goal is pathway interpretation (e.g. ranked multi-omics pathways) rather
  than unbiased predictive performance comparison — AUROC benchmarking alone does
  not validate biological findings
- 'When samples lack measurements across all omics views: PathIntegrate requires all
  input omics on the same individuals, so such samples must be discarded and CV design
  changes'
- When no labelled outcome exists (unsupervised integration), since AUROC evaluation
  is undefined
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PathIntegrate
  role: Provides the Multi-View (MB-PLS) and Single-View (PLS-DA on pathway scores)
    models being benchmarked
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Molecular-level and pathway-level baseline multi-omics integration model benchmarked
    against PathIntegrate
- name: SciKitLearn
  role: Supplies PLS-DA classifier, nested cross-validation machinery, and scoring;
    sspa fit/transform methods are designed for sklearn Pipeline/GridSearchCV compatibility
- name: py-ssPA
  role: Pathway transformation (ssPA) methods with fit/transform so pathway scores
    are computed fold-wise without test-set leakage
  repo: https://github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:b65fe804a8808bf89649b4db2be8e496d200944474c47ebdffd3a099dba8480a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - py-ssPA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8936170212765957
n_claims: 47
---

# cross-validated-auroc-benchmarking

## Summary

Benchmark predictive multi-omics models (e.g. PathIntegrate Multi-View/Single-View vs DIABLO) using repeated nested cross-validation with held-out test AUROC as the performance metric, tabulated per model and effect size.

## When to use

Use when comparing supervised multi-omics classifiers (molecular-level vs pathway-level) on semi-synthetic or real labelled data and you need unbiased generalisation performance reported as AUROC across effect sizes.

## When NOT to use

- When the goal is pathway interpretation (e.g. ranked multi-omics pathways) rather than unbiased predictive performance comparison — AUROC benchmarking alone does not validate biological findings
- When samples lack measurements across all omics views: PathIntegrate requires all input omics on the same individuals, so such samples must be discarded and CV design changes
- When no labelled outcome exists (unsupervised integration), since AUROC evaluation is undefined

## Inputs

- Semi-synthetic multi-omics DataFrames (samples as rows, molecule IDs as columns) at controlled effect sizes, e.g. COPDgene metabolomics and proteomics
- Binary class labels (e.g. COPD status)
- Pathway matrices from KEGG and Reactome downloaded via the sspa package
- Fitted model objects (PathIntegrate Multi-View MB-PLS, Single-View PLS-DA, DIABLO molecular and pathway-level)

## Outputs

- Held-out test AUROC per model, effect size, and pathway database
- Results table such as fig4A_auroc_results.csv
- Cross-validated tuned number of latent variables per model

## How to apply

Fit each competing model — PathIntegrate Multi-View (MB-PLS with a shared pathway-based latent space), PathIntegrate Single-View (multi-omics pathway scores into a scikit-learn PLS-DA classifier), DIABLO at the molecular level, and DIABLO on pathway scores — on semi-synthetic multi-omics data (e.g. COPDgene metabolomics and proteomics) generated at multiple controlled effect sizes. Tune the number of latent variables with 5-times repeated nested 5-fold cross-validation, then record the held-out test AUROC for every combination of model, effect size, and pathway database (KEGG and Reactome). The pathway transformation must be fit on training folds only and applied to test folds, which the sspa methods (sspa_kpca, sspa_SVD, sspa_ssClustPA) support via sklearn-compatible fit/transform. Tabulate test AUROC per effect size and per database comparison (e.g. fig4A_auroc_results.csv) rather than relying on training-fold metrics.

## Related tools

- **PathIntegrate** (Provides the Multi-View (MB-PLS) and Single-View (PLS-DA on pathway scores) models being benchmarked) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Molecular-level and pathway-level baseline multi-omics integration model benchmarked against PathIntegrate)
- **SciKitLearn** (Supplies PLS-DA classifier, nested cross-validation machinery, and scoring; sspa fit/transform methods are designed for sklearn Pipeline/GridSearchCV compatibility)
- **py-ssPA** (Pathway transformation (ssPA) methods with fit/transform so pathway scores are computed fold-wise without test-set leakage) — https://github.com/cwieder/py-ssPA

## Examples

```
from sklearn.model_selection import RepeatedStratifiedKFold; from sklearn.model_selection import cross_val_score; scores = cross_val_score(model, X, y, cv=RepeatedStratifiedKFold(n_splits=5, n_repeats=5), scoring='roc_auc')
```

## Evaluation signals

- Accepted if pathway-based models (PathIntegrate Multi-View, Single-View, 'DIABLO pathway') outperform the molecular-level 'DIABLO molecular-level' model in AUROC across all effect sizes
- KEGG pathway scores should show marginal AUROC advantage over Reactome at larger effect sizes
- Latent-variable tuning must use 5-times repeated nested 5-fold CV; verify inner-loop tuning is separated from outer-loop AUROC reporting
- ssPA transformation must be fit on training data and applied to test data (no leakage); verify via fit/transform usage
- Results table (e.g. fig4A_auroc_results.csv) contains one AUROC value per model × effect size × pathway database combination

## Limitations

- Semi-synthetic benchmarking signals may not reflect real differential abundance patterns; appropriate benchmarking data with known molecular and pathway-level signals was lacking
- Comparing molecular vs pathway-level models is confounded because the number of differentially abundant molecules per pathway is unknown and pathway definitions/sizes vary between databases
- Pathway coverage (proportion of pathway molecules observable) is a bottleneck, particularly for metabolomics
- All input omics must be measured on the same individuals; samples with missing entire omics views are discarded

## Evidence

- [supplementary] 5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [intro] PathIntegrate was benchmarked against DIABLO: "PathIntegrate was benchmarked against DIABLO [11], a popular multi-omics integration tool with a similar predictive framework"
- [readme] ssPA methods compatible with sklearn cross-validation: [evidence span withheld — anchor status: external_doc]
- [abstract] Grouping molecules into pathways benefits signal detection: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
