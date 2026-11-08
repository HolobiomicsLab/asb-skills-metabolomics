---
name: principal-curve-projection
description: Use when you have a metabolite abundance matrix (e.g., metabolite names
  or HMDB ids × samples) and need to reduce it to pathway-level features via PDS projection
  before feature selection or ML classification.
when_to_use_negative:
- Input is already a pathway-level or PDS feature matrix — re-running this skill would
  double-transform and misalign dimensions.
- You need to classify using raw metabolite intensities without pathway-based feature
  engineering; Lilikoi's classifier can take the original measurement matrix.
- The metabolites cannot be mapped to any pathway (MetaTOpathway returns an empty
  table), leaving no pathway to compute PDS over.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0154
tools:
- name: Lilikoi v2.0
  role: R package providing lilikoi.PDSfun to project metabolite measurements onto
    principal curves and compute the pathway dysregulation score matrix
  repo: github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which lilikoi.PDSfun and the surrounding Lilikoi
    v2.0 pipeline are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:3cdbde52c8c03b167a63ab99676875e9d4b64265228801ede6669637860c8780
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# principal-curve-projection

## Summary

Computes a pathway dysregulation score (PDS) for each sample-pathway pair in Lilikoi v2.0 by projecting metabolite intensities onto principal curves within each pathway, converting a raw metabolite abundance matrix into a normalized pathway-level feature matrix. This is the dimension-transformer step that turns metabolite-level measurements into personalized, pathway-based features for downstream feature selection and classification.

## When to use

Use when you have a metabolite abundance matrix (e.g., metabolite names or HMDB ids × samples) and need to reduce it to pathway-level features via PDS projection before feature selection or ML classification.

## When NOT to use

- Input is already a pathway-level or PDS feature matrix — re-running this skill would double-transform and misalign dimensions.
- You need to classify using raw metabolite intensities without pathway-based feature engineering; Lilikoi's classifier can take the original measurement matrix.
- The metabolites cannot be mapped to any pathway (MetaTOpathway returns an empty table), leaving no pathway to compute PDS over.

## Inputs

- Metabolite abundance matrix (samples × metabolites, e.g., plasma_breast_cancer.csv)
- Metabolite names or HMDB ids for pathway mapping
- Metabolite_pathway_table from lilikoi.MetaTOpathway

## Outputs

- PDS matrix (samples as rows, pathways as columns, each entry a pathway dysregulation score in [0,1])
- Pathway-level feature matrix for downstream feature selection and classification

## How to apply

Load the metabolite abundance matrix (e.g., plasma_breast_cancer.csv from the lilikoi package) and split it into Metadata and dataSet via lilikoi.Loaddata. Convert metabolite names to HMDB ids and map each metabolite to its associated metabolic pathway using lilikoi.MetaTOpathway, which returns a Metabolite_pathway_table. Pass that table to lilikoi.PDSfun, which projects metabolite measurements onto principal curves within each pathway to produce a pathway dysregulation score (PDS) per sample-pathway pair — each score lies in [0, 1], with larger values representing higher dysregulation. Assemble scores into a PDS matrix with samples as rows and pathways as columns (the dimension transformer output), then feed this matrix into lilikoi.featuresSelection for gain-ratio-based pathway selection. Rationale: personalized pathway-based representation is the highlight of Lilikoi; projecting onto principal curves captures the trajectory of metabolite variation within each pathway rather than treating metabolites as independent features.

## Related tools

- **Lilikoi v2.0** (R package providing lilikoi.PDSfun to project metabolite measurements onto principal curves and compute the pathway dysregulation score matrix) — github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which lilikoi.PDSfun and the surrounding Lilikoi v2.0 pipeline are executed)

## Examples

```
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)
```

## Evaluation signals

- Every PDS entry lies in [0, 1]; scores outside this range indicate a failed projection.
- PDS matrix shape: one row per sample and one column per pathway in the Metabolite_pathway_table.
- Larger PDS values correspond to higher pathway dysregulation — inspect score distribution relative to expected phenotype groups.
- The PDS matrix feeds directly into lilikoi.featuresSelection (threshold=0.50, method='gain'); a non-conformable matrix means the dimension transformation was not produced correctly.
- Pathways represented in the PDS matrix should match the pathway column of Metabolite_pathway_table.

## Limitations

- PDS computation depends on successful metabolite-to-pathway mapping via MetaTOpathway; unmapped metabolites are dropped and contribute no signal.
- Scores are normalized to [0,1] but the article does not describe per-pathway normalization details, so cross-dataset comparisons should be validated.
- The projection is pathway-local; it does not model metabolites outside the mapped pathway set.

## Evidence

- [readme] Compute PDS via principal-curve projection: "# Transform metabolites into pathway using pathtracer algorithm
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
- [other] PDS scores are per-sample pathway dysregulation values in [0,1]: [evidence span withheld — anchor status: not_found]
- [intro] Dimension transformation is one of Lilikoi's four core modules: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [readme] Load data and map metabolite names to pathways via MetaTOpathway: "convertResults=lilikoi.MetaTOpathway('name')
Metabolite_pathway_table = convertResults$table"
- [readme] Feed PDS matrix into feature selection: "selected_Pathways_Weka= lilikoi.featuresSelection(PDSmatrix,threshold= 0.50,method="gain")"
- [intro] PDS projection grounds the personalized, pathway-based representation: "Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi version 1 (v1)."
