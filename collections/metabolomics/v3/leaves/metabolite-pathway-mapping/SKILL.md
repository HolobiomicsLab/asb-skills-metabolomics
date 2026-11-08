---
name: metabolite-pathway-mapping
description: Use when you have a metabolite abundance matrix with metabolite names
  and need personalized pathway-level features for classification or survival analysis;
  also when you need a metabolite-pathway lookup table to select a specific pathway
  (e.
when_to_use_negative:
- Input is already a pathway-level feature table or PDS matrix — re-running MetaTOpathway/PDSfun
  would remap features incorrectly
- The analysis goal is feature selection, classification, or prognosis on an existing
  PDSmatrix — use those skills instead of this mapping step
- Metabolite identifiers are not recognized names that MetaTOpathway('name') can convert
  to HMDB ids
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3391
tools:
- name: Lilikoi v2.0
  role: Implements both lilikoi.MetaTOpathway (metabolite-name → HMDB id / pathway
    mapping) and lilikoi.PDSfun (PathTracer PDS computation) used by this skill.
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Runtime environment in which the Lilikoi v2.0 package functions are invoked
    to perform the metabolite-to-pathway transformation.
merged_aliases:
- metabolite-to-pathway-mapping
merged_alias_records:
- alias: metabolite-to-pathway-mapping
  slug: metabolite-to-pathway-mapping
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:64811c919565b75954ce93cdeda757d2af816256262128797578282e8b2816d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Cox-nnet
  - Lilikoi v2.0
  - R
  - lilikoi
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# metabolite-pathway-mapping

## Summary

Map metabolite identifiers to their associated metabolic pathways and transform metabolite-level abundance data into pathway-level features in Lilikoi v2.0. This is the dimension-transformation core of the personalized pathway-based analysis: raw metabolite measurements (e.g. plasma_breast_cancer.csv) become a normalized [0,1] pathway dysregulation score (PDS) matrix used for downstream classification and prognosis.

## When to use

Use when you have a metabolite abundance matrix with metabolite names and need personalized pathway-level features for classification or survival analysis; also when you need a metabolite-pathway lookup table to select a specific pathway (e.g. Pyruvate Metabolism) for regression or KEGG visualization.

## When NOT to use

- Input is already a pathway-level feature table or PDS matrix — re-running MetaTOpathway/PDSfun would remap features incorrectly
- The analysis goal is feature selection, classification, or prognosis on an existing PDSmatrix — use those skills instead of this mapping step
- Metabolite identifiers are not recognized names that MetaTOpathway('name') can convert to HMDB ids

## Inputs

- Metabolite abundance matrix with metabolite names as features and samples as rows (e.g. lilikoi.Loaddata output from plasma_breast_cancer.csv)
- Identifier type for the input metabolite names (passed as 'name' to lilikoi.MetaTOpathway)
- Optional pathway name for downstream subset/regression steps (e.g. 'Pyruvate Metabolism')

## Outputs

- Metabolite_pathway_table (lilikoi.MetaTOpathway convertResults$table) mapping each metabolite to its pathway(s)
- PDSmatrix from lilikoi.PDSfun: samples × pathways pathway dysregulation score matrix with every entry normalized to [0,1]

## How to apply

Call lilikoi.MetaTOpathway('name') to convert the metabolite names in the input dataset to HMDB ids and build a metabolite-pathway table; this produces the lookup used by every downstream pathway step. Feed that table into lilikoi.PDSfun, which implements the PathTracer algorithm to compute a per-sample pathway dysregulation score: for each pathway, metabolite intensities are projected onto principal curves yielding a score between 0 and 1, with larger PDS values representing higher dysregulation. Assemble the scores into a PDS matrix with samples as rows and pathways as columns, all normalized to [0,1]. The resulting PDSmatrix then drives feature selection (lilikoi.featuresSelection), classification, prognosis, and can be subset to a named pathway (e.g. 'Pyruvate Metabolism') for metabolite-pathway regression or KEGG plotting. Validate that the table and matrix exist, have the expected rows/columns, and that every score entry lies in [0,1].

## Related tools

- **Lilikoi v2.0** (Implements both lilikoi.MetaTOpathway (metabolite-name → HMDB id / pathway mapping) and lilikoi.PDSfun (PathTracer PDS computation) used by this skill.) — https://github.com/lanagarmire/lilikoi2
- **R** (Runtime environment in which the Lilikoi v2.0 package functions are invoked to perform the metabolite-to-pathway transformation.)

## Examples

```
convertResults=lilikoi.MetaTOpathway('name'); Metabolite_pathway_table=convertResults$table; PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)
```

## Evaluation signals

- Metabolite_pathway_table is produced by lilikoi.MetaTOpathway and is non-empty (head() shows mapped entries).
- PDSmatrix has samples as rows and pathways as columns.
- Every PDS entry falls within [0,1]; larger values represent higher dysregulation.
- Downstream skills (featuresSelection, machine_learning, prognosis, meta_path, KEGGplot) accept the PDSmatrix and Metabolite_pathway_table without re-mapping.
- A named pathway such as 'Pyruvate Metabolism' can be subset from Metabolite_pathway_table for regression or KEGG visualization.

## Limitations

- Mapping depends on metabolite names being resolvable to HMDB ids via MetaTOpathway('name'); unrecognized identifiers will fail to map to a pathway.
- The PDS is a per-sample, pathway-level summary — individual metabolite-level signals are collapsed, so metabolite-specific diagnostics must use the original dataset.
- Lilikoi v2.0 exposes no alternate identifier types in the provided context beyond the 'name' invocation shown.
- The resulting PDSmatrix depends on the pathway database used by PathTracer within Lilikoi; pathway definitions are not user-configurable in the documented workflow.

## Evidence

- [readme] MetaTOpathway converts metabolite names to HMDB ids and builds pathway table: "Transform the metabolite names to the HMDB ids using Lilikoi MetaTOpathway function"
- [readme] PDSfun transforms metabolites to pathway via pathtracer: [evidence span withheld — anchor status: external_doc]
- [other] PDS value range and dysregulation semantics: [evidence span withheld — anchor status: not_found]
- [other] PDS via principal curve projection: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: pathway-based representation and dimension transformer module: "Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi"
- [intro] Intro: four modules include feature mapper and dimension transformer: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
