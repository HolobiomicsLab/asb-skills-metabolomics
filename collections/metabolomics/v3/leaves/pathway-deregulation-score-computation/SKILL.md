---
name: pathway-deregulation-score-computation
description: Use when raw metabolite-level abundance data (e.g., named metabolites
  or HMDB ids) must be transformed into personalized pathway-based features before
  feature selection, machine learning classification, or Cox-PH/Cox-nnet prognosis
  prediction.
when_to_use_negative:
- Input is already a pathway-level feature matrix — re-running PDS transformation
  would be redundant
- Metabolites cannot be mapped to KEGG/HMDB pathways (e.g., unnamed or non-standard
  metabolite identifiers), so no pathway deregulation scores can be computed
- The task is pure preprocessing (normalization, imputation) or exploratory analysis
  (PCA, t-SNE) rather than pathway-based feature transformation
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: Lilikoi v2.0
  role: R package providing MetaTOpathway and PDSfun functions to map metabolites
    to pathways and compute the PDS matrix
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the Lilikoi v2.0 PDS computation is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:f724e2955628d154956e60ded90ef96293d6e7f0cb1fda9623b1a9db672b9957
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Cox-nnet
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pathway-deregulation-score-computation

## Summary

Computes pathway deregulation score (PDS) features by mapping metabolites to pathways (via HMDB ids and the pathtracer algorithm) and transforming a metabolite abundance matrix into a pathway-level feature matrix for downstream classification or prognosis modeling in Lilikoi v2.0.

## When to use

Use when raw metabolite-level abundance data (e.g., named metabolites or HMDB ids) must be transformed into personalized pathway-based features before feature selection, machine learning classification, or Cox-PH/Cox-nnet prognosis prediction.

## When NOT to use

- Input is already a pathway-level feature matrix — re-running PDS transformation would be redundant
- Metabolites cannot be mapped to KEGG/HMDB pathways (e.g., unnamed or non-standard metabolite identifiers), so no pathway deregulation scores can be computed
- The task is pure preprocessing (normalization, imputation) or exploratory analysis (PCA, t-SNE) rather than pathway-based feature transformation

## Inputs

- Metabolomics abundance matrix (metabolites x samples) with metabolite names, e.g., plasma_breast_cancer.csv loaded via lilikoi.Loaddata
- Patient survival times and event indicators when the PDS matrix feeds prognosis modeling

## Outputs

- Metabolite-to-pathway mapping table (Metabolite_pathway_table from MetaTOpathway)
- Pathway deregulation score (PDS) matrix (pathways x samples) for use in feature selection, machine learning, or Cox-PH/Cox-nnet models

## How to apply

First convert metabolite names to HMDB ids with lilikoi.MetaTOpathway('name'), which returns a metabolite-to-pathway mapping table (Metabolite_pathway_table). Then run lilikoi.PDSfun on that table to compute the pathway deregulation score matrix (PDSmatrix), which replaces metabolite columns with pathway-level features. This personalized, pathway-based representation is the hallmark of Lilikoi and feeds the feature selector (lilikoi.featuresSelection, e.g., threshold=0.50, method='gain') and downstream models. For prognosis tasks, the resulting PDS features are passed to lilikoi.prognosis; on the NCI breast tumor metabolomics dataset (67 tumors, 536 metabolites) this transformation plus Ridge-penalized Cox-PH and Cox-nnet yielded C-indexes of 0.64 and 0.66, respectively.

## Related tools

- **Lilikoi v2.0** (R package providing MetaTOpathway and PDSfun functions to map metabolites to pathways and compute the PDS matrix) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the Lilikoi v2.0 PDS computation is executed)

## Examples

```
PDSmatrix = lilikoi.PDSfun(Metabolite_pathway_table)
```

## Evaluation signals

- MetaTOpathway returns a non-empty metabolite-to-pathway table (Metabolite_pathway_table) and PDSfun returns a PDS matrix with pathway-level rows/columns
- Pathways retained after PDS computation survive feature selection at the chosen threshold (e.g., lilikoi.featuresSelection threshold=0.50, method='gain')
- On the NCI breast tumor dataset (67 tumors, 536 metabolites), downstream Cox-PH should reach C-index ~0.64 (log-rank P = 0.04) and Cox-nnet ~0.66 (log-rank P = 0.02)
- The PDS matrix dimensions match the number of samples and mapped pathways, and sample labels (Metadata$Label) align with the original input

## Limitations

- PDS computation depends on successful metabolite-to-HMDB/pathway mapping; metabolites that cannot be converted are dropped from pathway-level representation
- Reported performance (C-index 0.64/0.66) is specific to pathway-transformed NCI breast tumor metabolomics data; do not expect transfer of those metrics to other datasets
- Downstream Cox-nnet modeling may require a Python installation (python.path parameter in lilikoi.prognosis)

## Evidence

- [intro] Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi version 1 (v1).: "Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi version 1 (v1)."
- [intro] It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [readme] Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function: "# Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function
convertResults=lilikoi.MetaTOpathway('name')"
- [readme] Transform metabolites into pathway using pathtracer algorithm: "# Transform metabolites into pathway using pathtracer algorithm
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
- [results] On the NCI breast tumor dataset, the Cox-PH model achieved a C-index of 0.64 with log-rank P = 0.04, and the Cox-nnet model performed slightly better with a C-index of 0.66 and log-rank P = 0.02.: [evidence span withheld — anchor status: not_found]
