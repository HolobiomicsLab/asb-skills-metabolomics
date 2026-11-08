---
name: plage-svd-pathway-decomposition
description: Use when molecular multi-omics abundance data must be converted to per-sample
  pathway activity scores via an unsupervised linear decomposition, as the alternative
  to kPCA scoring in PathIntegrate.
when_to_use_negative:
- The input is already a pathway-level feature table — no ssPA transformation (kPCA,
  SVD/PLAGE, or ssClustPA) is needed before PathIntegrate modelling.
- Rows (samples) contain missing values that cannot be imputed; the README pre-processing
  removes metabolites with too many NAs (columns with >50% NA/0/space are dropped)
  and median-imputes the remainder before SVD/PLAGE scoring, and PathIntegrate discards
  individuals lacking data on all omics.
- Only un-annotated molecules (no ChEBI/UniProt/ENSEMBL or KEGG mapping) are present
  — pathway-based scoring cannot incorporate them, causing information loss as noted
  in the discussion.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3557
tools:
- name: sspa
  role: Python package providing the PLAGE/SVD single-sample pathway scoring method
    (sspa_SVD) used by PathIntegrate to transform molecular abundance data into pathway-level
    scores.
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Downstream framework that consumes the SVD/PLAGE-derived pathway-score matrix
    as input to Single-View (SciKitLearn classifier/regressor) or Multi-View (MB-PLS)
    integration models.
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: 'Compatibility layer: SVD/PLAGE ssPA objects expose fit/transform/fit_transform
    so they integrate with SKlearn Pipeline and GridSearchCV for model optimisation
    and evaluation.'
  repo: https://github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:1871d50bda98eefbca8c2a56d75026f877e919ef0db818fab7674ec5ecc604fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3557
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# plage-svd-pathway-decomposition

## Summary

PLAGE/SVD single-sample pathway analysis decomposes each pathway's molecular abundance matrix via SVD to yield PC1-derived pathway activity scores per sample, one of the two ssPA transformations PathIntegrate applies before supervised multi-omics integration.

## When to use

Use when molecular multi-omics abundance data must be converted to per-sample pathway activity scores via an unsupervised linear decomposition, as the alternative to kPCA scoring in PathIntegrate.

## When NOT to use

- The input is already a pathway-level feature table — no ssPA transformation (kPCA, SVD/PLAGE, or ssClustPA) is needed before PathIntegrate modelling.
- Rows (samples) contain missing values that cannot be imputed; the README pre-processing removes metabolites with too many NAs (columns with >50% NA/0/space are dropped) and median-imputes the remainder before SVD/PLAGE scoring, and PathIntegrate discards individuals lacking data on all omics.
- Only un-annotated molecules (no ChEBI/UniProt/ENSEMBL or KEGG mapping) are present — pathway-based scoring cannot incorporate them, causing information loss as noted in the discussion.

## Inputs

- z-score-standardised molecular abundance matrix (pandas DataFrame, samples x molecules, with ChEBI/UniProt/ENSEMBL or KEGG compound/gene identifiers as column names)
- sspa pathway DataFrame (pathways as rows, molecules as columns) from sspa.process_reactome, sspa.process_kegg, or sspa.process_gmt
- minimum pathway size threshold (min_entity, e.g. 2) controlling which pathways are scored

## Outputs

- N x P pathway activity-score matrix (pandas DataFrame, samples as index, pathway IDs as columns) written to pathway_scores.csv
- Fitted SVD/PLAGE model object compatible with SciKitLearn fit/transform (usable in Pipeline and GridSearchCV)

## How to apply

Take the z-score-standardised molecular abundance matrix (samples as rows, ChEBI/UniProt/ENSEMBL for Reactome or KEGG compound/gene IDs for KEGG as columns) and the pathway DataFrame from the sspa package (pathways as rows, molecules as columns). For each pathway, extract the sub-matrix of molecules belonging to that pathway and apply singular value decomposition (SVD); the scores on the first principal component serve as the per-sample pathway activity score, following the PLAGE (Pathway Level Analysis of Gene Expression) approach. In practice call sspa.sspa_SVD (or sspa.sspa_svd) with a min_entity threshold (e.g. min_entity=2) to skip pathways too small to decompose, and use fit_transform so the SVD model is fit on training data and reused to transform test data — the README notes that for sspa.sspa_SVD 'the model can be fit on the training data and the test data is transformed using the fitted model'. PathIntegrate applies this transformation to each omics view independently, then concatenates the resulting N x P pathway-score matrices across omics for Single-View modelling or feeds them as blocks to MB-PLS for Multi-View. The resulting pathway_scores.csv has sample IDs as index and pathway IDs as columns and must contain finite (non-missing) scores for every pathway with molecules observed in the data.

## Related tools

- **sspa** (Python package providing the PLAGE/SVD single-sample pathway scoring method (sspa_SVD) used by PathIntegrate to transform molecular abundance data into pathway-level scores.) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Downstream framework that consumes the SVD/PLAGE-derived pathway-score matrix as input to Single-View (SciKitLearn classifier/regressor) or Multi-View (MB-PLS) integration models.) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Compatibility layer: SVD/PLAGE ssPA objects expose fit/transform/fit_transform so they integrate with SKlearn Pipeline and GridSearchCV for model optimisation and evaluation.) — https://github.com/cwieder/py-ssPA

## Examples

```
kpca_or_svd_scores = sspa.sspa_SVD(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- The output matrix has exactly one row per input sample and one column per scored pathway (samples as index, pathway IDs as columns).
- All scored pathways containing molecules observed in the data yield finite (non-missing) score values — no NaNs in pathway_scores.csv.
- The SVD model was fit on training data and reused to transform held-out test data (fit/transform pattern), not refit on the test set.
- Pathways below the min_entity threshold (e.g. fewer than 2 observed molecules) are excluded from the output rather than scored.
- The same input sample set produces consistent scores under repeated fit_transform calls given identical standardised input and pathway definitions.

## Limitations

- Pathway coverage is an inherent bottleneck: only the proportion of molecules in a pathway observable in the data can contribute to the score, which is particularly problematic for metabolomics.
- Reliance on current pathway annotation knowledge is a widely accepted limitation; un-annotated molecules cannot be incorporated and cause information loss.
- PLAGE/SVD is the linear alternative to kernel-PCA ssPA; when pathway–phenotype relationships are non-linear, kPCA may capture them better.
- PathIntegrate currently does not support samples with entire rows of missing data, so individuals without all omics must be discarded before scoring.

## Evidence

- [methods] PLAGE method (SVD-derived PC1 scores) as the alternative pathway activity-score matrix: [evidence span withheld — anchor status: not_found]
- [methods] ssPA combines molecular abundances into pathway-level matrices: [evidence span withheld — anchor status: not_found]
- [readme] SVD/PLAGE model fit on training data and transformed on test data: [evidence span withheld — anchor status: external_doc]
- [readme] sspa package provides single-sample pathway analysis methods compatible with SciKitLearn: [evidence span withheld — anchor status: external_doc]
- [abstract] PathIntegrate transforms molecular data to pathway-level via ssPA: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level, and applies a predictive single-view or multi-view model to integrate"
- [readme] Pre-processing before ssPA: remove NAs, impute, log transform, z-score: [evidence span withheld — anchor status: external_doc]
