---
name: pathway-dysregulation-scoring
description: Use when a metabolomics abundance matrix (metabolites as features) must
  be converted into a pathway-level feature matrix before feature selection or disease
  classification in Lilikoi v2.0.
when_to_use_negative:
- Input is already a pathway-level feature table (e.g., a prior PDS matrix) — re-running
  PDS would double-transform features.
- The goal is metabolite-pathway regression or pathway visualization (use lilikoi.meta_path
  or lilikoi.KEGGplot instead), not dimension transformation.
- Metabolite names cannot be mapped to pathways (MetaTOpathway conversion fails or
  yields an empty table), leaving no metabolite-pathway associations to project.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Lilikoi v2.0
  role: R package supplying lilikoi.MetaTOpathway (metabolite name to HMDB id/pathway
    mapping) and lilikoi.PDSfun (pathtracer/principal-curve-based PDS computation)
    used for this transformation
  repo: github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which Lilikoi v2.0 and the PDS transformation are
    executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:f08e0d2a136032d7b84f21d453cd435580e5bd4a388a048a9023d7b2f9f87133
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pathway-dysregulation-scoring

## Summary

Per-sample dimension transformation in Lilikoi v2.0 that projects metabolite-level intensities onto pathway-level features by computing a pathway dysregulation score (PDS) between 0 and 1 for every sample-pathway pair, producing a samples x pathways PDS matrix.

## When to use

Use when a metabolomics abundance matrix (metabolites as features) must be converted into a pathway-level feature matrix before feature selection or disease classification in Lilikoi v2.0.

## When NOT to use

- Input is already a pathway-level feature table (e.g., a prior PDS matrix) — re-running PDS would double-transform features.
- The goal is metabolite-pathway regression or pathway visualization (use lilikoi.meta_path or lilikoi.KEGGplot instead), not dimension transformation.
- Metabolite names cannot be mapped to pathways (MetaTOpathway conversion fails or yields an empty table), leaving no metabolite-pathway associations to project.

## Inputs

- Metabolite abundance matrix (CSV, samples as rows, metabolite names as columns; e.g., Lilikoi's built-in plasma_breast_cancer.csv) with accompanying Metadata from lilikoi.Loaddata
- Metabolite-to-pathway mapping table (Metabolite_pathway_table) returned by lilikoi.MetaTOpathway after name-to-HMDB-id conversion

## Outputs

- PDS matrix (samples as rows, pathways as columns) with all pathway dysregulation scores normalized to [0,1], written to pds_matrix.csv

## How to apply

Load the abundance matrix (e.g., the built-in plasma_breast_cancer.csv via lilikoi.Loaddata) and first run lilikoi.MetaTOpathway('name') to map metabolite names to HMDB ids and their associated metabolic pathways, yielding Metabolite_pathway_table. Then call lilikoi.PDSfun on that table (the README describes this as transforming metabolites into pathways using the pathtracer algorithm, i.e., projecting mapped metabolite measurements onto principal curves within each pathway). Collect the resulting dysregulation scores into a PDS matrix with samples as rows and pathways as columns, normalized so every score lies in [0,1] — larger PDS values represent higher dysregulation. The rationale for this personalized, pathway-based representation is that it is the defining feature distinguishing Lilikoi from other metabolomics analysis packages. Write the matrix to pds_matrix.csv for downstream lilikoi.featuresSelection and lilikoi.machine_learning steps.

## Related tools

- **Lilikoi v2.0** (R package supplying lilikoi.MetaTOpathway (metabolite name to HMDB id/pathway mapping) and lilikoi.PDSfun (pathtracer/principal-curve-based PDS computation) used for this transformation) — github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which Lilikoi v2.0 and the PDS transformation are executed)

## Examples

```
convertResults=lilikoi.MetaTOpathway('name'); PDSmatrix=lilikoi.PDSfun(convertResults$table)
```

## Evaluation signals

- Output pds_matrix.csv exists and parses as a matrix with one row per sample and one column per pathway.
- Every PDS entry falls within the [0,1] interval.
- Number of rows in the PDS matrix equals the number of samples in the input abundance matrix/Metadata.
- Columns correspond to the pathways returned by the Metabolite_pathway_table mapping from lilikoi.MetaTOpathway.
- Downstream lilikoi.featuresSelection can consume the PDS matrix (e.g., threshold=0.50, method='gain') without schema errors.

## Limitations

- PDS coverage depends on successful metabolite-to-pathway mapping; metabolites that MetaTOpathway cannot convert to HMDB ids/pathways are not represented in the pathway features.
- Scores are relative dysregulation measures (0-1) derived via principal-curve/pathtracer projection, not absolute metabolite concentrations, and should be interpreted as pathway-level features.
- The transformation is coupled to Lilikoi's pathway database and module ordering (feature mapper -> dimension transformer -> feature selector -> classification predictor); using PDS output outside this pipeline is untested.

## Evidence

- [intro] dimension transformer as Lilikoi module: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [intro] pathway-based representation as Lilikoi highlight: "Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi version 1 (v1)."
- [readme] MetaTOpathway name-to-HMDB conversion: "# Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function"
- [readme] PDSfun pathtracer transformation: "# Transform metabolites into pathway using pathtracer algorithm
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
