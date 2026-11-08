---
name: rweka-attribute-evaluation
description: Use when you have a pathways x samples pathway deregulation score (PDS)
  matrix from lilikoi.get_pd_scores and need to filter and rank pathway features against
  a phenotype Label column before classification.
when_to_use_negative:
- Input is still a raw metabolite-sample matrix — run the dimension transformation
  module (lilikoi.get_pd_scores) first; this skill expects a pathway-level PDS matrix
- You need group-level pathway enrichment rather than individual-sample pathway features
  for classification (MetPA, IMPaLA, MPEA operate at group level)
- Features have already been selected and a trained model exists — use lilikoi.adjust_model
  instead of re-running attribute evaluation
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lilikoi
  role: Provides lilikoi.select_pathways, the wrapper that applies RWeka attribute
    evaluation to the PDS matrix
  repo: https://github.com/lanagarmire/lilikoi
- name: RWeka
  role: Backend implementing the information-gain (and gain-ratio) attribute evaluation
    used to score each pathway feature against sample labels
- name: R
  role: Language in which the feature selection module runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:4c8c97dc66fa00a4742d1d4486f8e417f956f513a9b4d87f4040929b961f3a03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - RWeka
  - lilikoi
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# RWeka attribute evaluation

## Summary

Ranks pathway-level features in Lilikoi's PDS score matrix by information gain (mutual information) via RWeka's attribute evaluation, so that only pathways most relevant to the disease phenotype (e.g. breast cancer vs normal label) feed into the classifier.

## When to use

Use when you have a pathways x samples pathway deregulation score (PDS) matrix from lilikoi.get_pd_scores and need to filter and rank pathway features against a phenotype Label column before classification.

## When NOT to use

- Input is still a raw metabolite-sample matrix — run the dimension transformation module (lilikoi.get_pd_scores) first; this skill expects a pathway-level PDS matrix
- You need group-level pathway enrichment rather than individual-sample pathway features for classification (MetPA, IMPaLA, MPEA operate at group level)
- Features have already been selected and a trained model exists — use lilikoi.adjust_model instead of re-running attribute evaluation

## Inputs

- PDS score matrix (pathways x samples) from lilikoi.get_pd_scores
- metabolite measurement table (plasma_breast_cancer.csv) with a phenotype Label column
- threshold value (e.g. 0.42) and method string ('gain' or 'gainratio')

## Outputs

- Ranked table of selected pathway identifiers and their information-gain scores (e.g. selected_pathways.csv)
- significantPathways object passed to lilikoi.machine_learning

## How to apply

Load the PDS matrix produced by lilikoi.get_pd_scores together with the metabolite measurement table (plasma_breast_cancer.csv) that contains the phenotype Label column, then call lilikoi.select_pathways with threshold = 0.42 and method = "gain", which internally invokes RWeka's information-gain attribute evaluation. Each pathway is scored with a relevance value between 0 and 1; retain only features whose gain score exceeds the 0.42 threshold and rank them in descending order of gain. When the dataset contains additional categorical variables (e.g. Age, Race from plasma_breast_cancer_Meta.csv), switch method = "gainratio" to use gain ratio as the alternative statistic. Pass the resulting ranked pathway set into lilikoi.machine_learning for model building.

## Related tools

- **lilikoi** (Provides lilikoi.select_pathways, the wrapper that applies RWeka attribute evaluation to the PDS matrix) — https://github.com/lanagarmire/lilikoi
- **RWeka** (Backend implementing the information-gain (and gain-ratio) attribute evaluation used to score each pathway feature against sample labels)
- **R** (Language in which the feature selection module runs)

## Examples

```
significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain")
```

## Evaluation signals

- The run completes without error and selected_pathways.csv exists with a pathway identifier column and a gain-score column
- Every retained pathway feature has an information-gain score above the 0.42 threshold
- Gain scores fall in the expected range of 0 to 1
- Ranked table is sorted in descending order of gain
- Feeding the selected pathways to lilikoi.machine_learning succeeds without missing-feature errors

## Limitations

- The 0.42 threshold is dataset-dependent and must be retuned for new cohorts
- Information gain is the default statistic; when extra categorical covariates (e.g. Age, Race) are present, gain ratio should be used instead
- About 20% of metabolites cannot be mapped to standard names (Levenshtein-distance matching), which shrinks the pathway features available for selection

## Evidence

- [methods] significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain"): "significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain")"
- [readme] selects the pathways with the highest mutual information and then uses them to build a classifier.: "selects the pathways with the highest mutual information and then uses them to build a classifier."
- [abstract] feature selection module, which helps to select the significant pathway features related to the disease phenotypes: "feature selection module, which helps to select the significant pathway features related to the disease phenotypes"
