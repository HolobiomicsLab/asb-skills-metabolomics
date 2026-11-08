---
name: pathway-level-feature-selection
description: Use when you have a per-sample pathway score matrix (e.g., Lilikoi PDS
  matrix built by lilikoi.PDSfun) and need to pick the most significant pathway features
  before running a classification or prognosis model in Lilikoi v2.0.
when_to_use_negative:
- Input is still raw metabolite intensities rather than a pathway-level (PDS) score
  matrix — run lilikoi.MetaTOpathway and lilikoi.PDSfun first, since featuresSelection
  expects pathway-level features.
- You want a survival-feature selection step for the Cox-PH / Cox-nnet prognosis module
  instead of the disease-classification pathway filter — the article describes featuresSelection
  in the classification workflow, not for prognosis feature picking.
- The sample labels are unavailable or unreliable — information-gain selection is
  supervised and needs phenotype labels to rank pathways.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Lilikoi v2.0
  role: R package whose lilikoi.featuresSelection function performs the information-gain
    pathway feature selection (threshold=0.50) on the PDS matrix
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which lilikoi.featuresSelection is invoked on the
    PDS matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:7e16cc2ca291200dd122a95e0cd3b66409c3b37cfadd7e053a63a84ffa146b9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  - lilikoi
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- pathway-feature-ranking
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pathway-level-feature-selection

## Summary

Selects the phenotype-informative pathways from a metabolite-derived pathway dysregulation score (PDS) matrix using information gain ranking with a 0.50 threshold, as implemented in Lilikoi v2.0's lilikoi.featuresSelection, so that downstream classification/regression operates on a reduced set of significant pathways.

## When to use

Use when you have a per-sample pathway score matrix (e.g., Lilikoi PDS matrix built by lilikoi.PDSfun) and need to pick the most significant pathway features before running a classification or prognosis model in Lilikoi v2.0.

## When NOT to use

- Input is still raw metabolite intensities rather than a pathway-level (PDS) score matrix — run lilikoi.MetaTOpathway and lilikoi.PDSfun first, since featuresSelection expects pathway-level features.
- You want a survival-feature selection step for the Cox-PH / Cox-nnet prognosis module instead of the disease-classification pathway filter — the article describes featuresSelection in the classification workflow, not for prognosis feature picking.
- The sample labels are unavailable or unreliable — information-gain selection is supervised and needs phenotype labels to rank pathways.

## Inputs

- Pathway dysregulation score (PDS) matrix: pathway-level scores per sample, produced by lilikoi.PDSfun from the metabolite-pathway mapping table (Metabolite_pathway_table)
- Phenotype labels per sample (e.g., Metadata$Label from lilikoi.Loaddata on an input CSV such as plasma_breast_cancer.csv)

## Outputs

- selected_Pathways_Weka: the ranked list of pathways passing the 0.50 information-gain threshold
- selected_pathways.csv: persisted CSV of the selected pathway features

## How to apply

In R, load the PDS matrix produced by the dimension-transformation step (lilikoi.PDSfun applied to the metabolite-pathway table from lilikoi.MetaTOpathway) and pass it to lilikoi.featuresSelection with method='gain' (information-gain ranking, the Weka-style InfoGainAttributeEval+Ranker approach) and threshold=0.50. The function scores each pathway's information gain with respect to the phenotype labels (e.g., case/control in Metadata$Label) and retains only pathways whose gain clears the 0.50 cutoff. Decision point: pathways below threshold are dropped rather than kept at reduced weight, so the number of selected pathways is data-dependent and shrinks with a higher threshold. Write the retained set to selected_pathways.csv and carry the returned selected_Pathways_Weka object forward into lilikoi.machine_learning (as significantPathways) and lilikoi.meta_path regression. Rationale: operating at the pathway level rather than metabolite level gives personalized, pathway-based features, and a conservative gain threshold avoids feeding noisy pathways into classifiers such as SVM, GBM, RF, or the new deep learning model.

## Related tools

- **Lilikoi v2.0** (R package whose lilikoi.featuresSelection function performs the information-gain pathway feature selection (threshold=0.50) on the PDS matrix) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which lilikoi.featuresSelection is invoked on the PDS matrix)

## Examples

```
selected_Pathways_Weka = lilikoi.featuresSelection(PDSmatrix, threshold=0.50, method="gain")
```

## Evaluation signals

- selected_Pathways_Weka is a non-empty ranked subset of PDS matrix row names (pathways), with each retained pathway's information gain >= 0.50.
- The number of selected pathways is strictly less than the total number of pathways in the PDS matrix, confirming the threshold actually filtered features.
- The selected pathway names all exist in the upstream Metabolite_pathway_table mapping (no orphan features), preserving traceability from metabolite to pathway.
- Downstream lilikoi.machine_learning call receives selected_Pathways_Weka (via significantPathways) without error, confirming schema compatibility.
- selected_pathways.csv round-trips: re-loading it reproduces the same pathway list passed to classification.

## Limitations

- The 0.50 information-gain threshold is a fixed default in the Lilikoi workflow; datasets with weak signal may yield very few or no selected pathways, and the README example does not describe an automatic fallback.
- Selection is supervised on the phenotype labels, so it cannot be used in label-free/exploratory analyses and risks overfitting if labels are noisy.
- Feature selection operates on pathways only after the metabolite-to-HMDB mapping and PDS transformation; metabolites that fail name mapping (name or HMDB) are implicitly lost upstream.

## Evidence

- [methods] Pathway feature selection is performed by lilikoi.featuresSelection on the PDS matrix, using the information-gain method with a threshold of 0.50, returning the selected pathways as selected_Pathways_Weka.: [evidence span withheld — anchor status: not_found]
- [readme] README feature-selection call: "selected_Pathways_Weka= lilikoi.featuresSelection(PDSmatrix,threshold= 0.50,method="gain")"
- [intro] Article intro module list: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [readme] README upstream PDS construction: "PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
