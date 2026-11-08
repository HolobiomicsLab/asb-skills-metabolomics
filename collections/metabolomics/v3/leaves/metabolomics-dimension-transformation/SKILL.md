---
name: metabolomics-dimension-transformation
description: Use when a raw metabolite abundance matrix (e.g. plasma_breast_cancer.csv
  with metabolite names as features) must be converted into a per-sample pathway dysregulation
  score (PDS) matrix before feature selection or classification in Lilikoi.
when_to_use_negative:
- The input is already a pathway-level PDS matrix or a feature table after feature
  selection — rerunning the dimension transformer would double-transform the features.
- The analysis goal is prognosis or raw metabolite-level regression (e.g. lilikoi.prognosis
  or lilikoi.meta_path on metabolite data) that does not require pathway-level projection
  first.
- Only exploratory preprocessing/visualization (PCA, t-SNE, normalization, imputation)
  is needed — those are separate Lilikoi modules and do not require PDS transformation.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: lilikoi.MetaTOpathway
  role: maps metabolite names to HMDB ids and their associated metabolic pathways,
    producing the Metabolite_pathway_table that PDSfun consumes
  repo: https://github.com/lanagarmire/lilikoi2
- name: lilikoi.PDSfun
  role: computes the pathway dysregulation score matrix using the pathtracer algorithm,
    the actual dimension transformation step
  repo: https://github.com/lanagarmire/lilikoi2
- name: Lilikoi v2.0
  role: the R package whose dimension transformer module implements this metabolite-to-pathway
    transformation
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: programming environment in which Lilikoi v2.0 and this transformation run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:7e93086ce061c352396711f8beeed22e2b2d0197892bfd394d40b1717d1088a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  - lilikoi
  - lilikoi.MetaTOpathway
  - lilikoi.PDSfun
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-matrix-transformation
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metabolomics-dimension-transformation

## Summary

Transforms a metabolite-level abundance matrix into pathway-level features by computing a pathway dysregulation score (PDS) for each sample-pathway pair, as implemented in the Lilikoi v2.0 dimension transformer module. This per-sample projection is what makes downstream feature selection and disease classification pathway-based rather than metabolite-based.

## When to use

Use when a raw metabolite abundance matrix (e.g. plasma_breast_cancer.csv with metabolite names as features) must be converted into a per-sample pathway dysregulation score (PDS) matrix before feature selection or classification in Lilikoi.

## When NOT to use

- The input is already a pathway-level PDS matrix or a feature table after feature selection — rerunning the dimension transformer would double-transform the features.
- The analysis goal is prognosis or raw metabolite-level regression (e.g. lilikoi.prognosis or lilikoi.meta_path on metabolite data) that does not require pathway-level projection first.
- Only exploratory preprocessing/visualization (PCA, t-SNE, normalization, imputation) is needed — those are separate Lilikoi modules and do not require PDS transformation.

## Inputs

- metabolite abundance matrix (samples x metabolites), e.g. plasma_breast_cancer.csv loaded via lilikoi.Loaddata
- metabolite-to-pathway mapping table (Metabolite_pathway_table) returned by lilikoi.MetaTOpathway('name')

## Outputs

- PDS matrix: samples as rows, pathways as columns, each entry a pathway dysregulation score normalized to [0,1]
- pds_matrix.csv (written PDS matrix, if persisted)

## How to apply

Load the metabolomics abundance matrix with lilikoi.Loaddata (e.g. the plasma_breast_cancer.csv from the package extdata), then run lilikoi.MetaTOpathway('name') to map each metabolite name to its associated metabolic pathway and obtain a Metabolite_pathway_table. Next call lilikoi.PDSfun(Metabolite_pathway_table), which uses the pathtracer algorithm to project the mapped metabolite measurements onto principal curves within each pathway, yielding a pathway dysregulation score DP(i) bounded in [0,1] per sample-pathway pair. Assemble the resulting scores into a samples-by-pathways PDS matrix; larger PDS values represent higher pathway dysregulation. The PDS matrix is then the direct input to lilikoi.featuresSelection (e.g. with method='gain' and threshold=0.50) and to lilikoi.machine_learning.

## Related tools

- **lilikoi.MetaTOpathway** (maps metabolite names to HMDB ids and their associated metabolic pathways, producing the Metabolite_pathway_table that PDSfun consumes) — https://github.com/lanagarmire/lilikoi2
- **lilikoi.PDSfun** (computes the pathway dysregulation score matrix using the pathtracer algorithm, the actual dimension transformation step) — https://github.com/lanagarmire/lilikoi2
- **Lilikoi v2.0** (the R package whose dimension transformer module implements this metabolite-to-pathway transformation) — https://github.com/lanagarmire/lilikoi2
- **R** (programming environment in which Lilikoi v2.0 and this transformation run)

## Examples

```
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)
```

## Evaluation signals

- Output PDS matrix has one row per sample and one column per pathway, with column count equal to the number of distinct pathways in the Metabolite_pathway_table.
- Every entry in the PDS matrix lies within the [0,1] interval, as PDS values are normalized to that range.
- pds_matrix.csv exists and parses as a numeric matrix without missing values for transformed samples.
- Larger PDS values correspond to higher dysregulation, consistent with the paper's definition of DP(i).
- The PDS matrix can be passed directly to lilikoi.featuresSelection(PDSmatrix, threshold=0.50, method='gain') without further reshaping.

## Limitations

- PDS scores are bounded to [0,1], so pathways with small dynamic ranges in the raw metabolite data may compress toward one end of the scale.
- The transformation depends on lilikoi.MetaTOpathway's metabolite-name-to-pathway mapping; metabolite names not recognized in the built-in pathway database will not contribute to any pathway score.
- PDS values are computed per pathway using principal curves within each pathway, so the score reflects within-pathway dysregulation and is not directly comparable to a raw metabolite intensity.

## Evidence

- [intro] PDS computation grounds: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [readme] README pipeline sequence: "# Transform metabolites into pathway using pathtracer algorithm
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
- [readme] README mapping step: "# Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function
convertResults=lilikoi.MetaTOpathway('name')
Metabolite_pathway_table = convertResults$table"
- [readme] threshold for downstream feature selection: "selected_Pathways_Weka= lilikoi.featuresSelection(PDSmatrix,threshold= 0.50,method="gain")"
- [intro] feature-mapping rationale: "Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi version 1 (v1)."
