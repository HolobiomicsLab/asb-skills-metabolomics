---
name: metabolite-kegg-identifier-mapping
description: Use when a metabolomics feature table uses plain compound names (or HMDB/KEGG/PUBCHEM/ChEBI
  IDs) instead of a normalized identifier, and you need to map metabolites to HMDB
  IDs and KEGG pathways before running Lilikoi v2.0's pathway-based downstream analyses.
when_to_use_negative:
- The input feature table is already a pathway-level matrix (e.g., PDSmatrix) — skip
  straight to feature selection or regression rather than re-mapping identifiers.
- The metabolites cannot be matched to any of the supported identifier vocabularies
  (name, HMDB, KEGG, PUBCHEM, ChEBI) or are custom compounds absent from the mapping
  databases, since the conversion would return no valid HMDBID/pathwayID rows.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Lilikoi v2.0
  role: Provides the lilikoi.MetaTOpathway function that converts metabolite names
    or IDs into HMDB identifiers and a metabolite-pathway table used for all subsequent
    Lilikoi pathway analyses.
  repo: https://github.com/lanagarmire/lilikoi2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:1799b6853da48996502127ff6838e71912caf5744389e1105781e70ee47724d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - KEGG
  - KEGGREST
  - Lilikoi v2.0
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# metabolite-kegg-identifier-mapping

## Summary

Use Lilikoi v2.0's MetaTOpathway function to convert raw metabolite names (or other identifiers such as HMDB IDs, KEGG IDs, PUBCHEM IDs, or ChEBI IDs) into HMDB identifiers and a metabolite-pathway table, which is the prerequisite for Lilikoi's pathway-based analyses (PDSmatrix, feature selection, machine learning, prognosis, and metabolite-pathway regression).

## When to use

Use when a metabolomics feature table uses plain compound names (or HMDB/KEGG/PUBCHEM/ChEBI IDs) instead of a normalized identifier, and you need to map metabolites to HMDB IDs and KEGG pathways before running Lilikoi v2.0's pathway-based downstream analyses.

## When NOT to use

- The input feature table is already a pathway-level matrix (e.g., PDSmatrix) — skip straight to feature selection or regression rather than re-mapping identifiers.
- The metabolites cannot be matched to any of the supported identifier vocabularies (name, HMDB, KEGG, PUBCHEM, ChEBI) or are custom compounds absent from the mapping databases, since the conversion would return no valid HMDBID/pathwayID rows.

## Inputs

- Metabolomics feature table (R data frame / CSV) with a first column of metabolite names or identifiers (name, HMDBID, KEGGID, PUBCHEM, or ChEBI)
- Identifier type string passed to lilikoi.MetaTOpathway (e.g., 'name')

## Outputs

- Conversion results object (convertResults) containing the metabolite-pathway table (Metabolite_pathway_table) with metabolite-to-pathway assignments
- Metabolite-pathway table with HMDBID and pathwayID columns used as input to lilikoi.PDSfun, feature selection, and pathway regression

## How to apply

Load the raw metabolomics data into R (e.g., via lilikoi.Loaddata on the plasma_breast_cancer.csv example or user-supplied CSV) and then call lilikoi.MetaTOpathway('name') when the first column contains metabolite names, or lilikoi.MetaTOpathway('HMDBID'/'KEGGID'/'PUBCHEM'/'ChEBI') for other identifier types. This returns a conversion result object from which the metabolite-pathway table is extracted via convertResults$table, yielding columns such as HMDBID and pathwayID that link each metabolite to pathway memberships. The rationale for mapping to HMDB IDs is that Lilikoi's subsequent steps (pathtracer-based PDS transformation, feature selection with gain ratio, KEGG plot, and metabolite-pathway regression) operate on pathway identifiers derived from these normalized compound identifiers. Validate the mapping by inspecting head(Metabolite_pathway_table) for correct HMDBID/pathwayID pairs and by confirming the subsequent lilikoi.PDSfun call produces a pathway-level matrix without unmapped-metabolite warnings.

## Related tools

- **Lilikoi v2.0** (Provides the lilikoi.MetaTOpathway function that converts metabolite names or IDs into HMDB identifiers and a metabolite-pathway table used for all subsequent Lilikoi pathway analyses.) — https://github.com/lanagarmire/lilikoi2

## Examples

```
convertResults <- lilikoi.MetaTOpathway('name'); Metabolite_pathway_table <- convertResults$table
```

## Evaluation signals

- head(Metabolite_pathway_table) shows expected HMDBID and pathwayID columns populated for the majority of input metabolites.
- The subsequent lilikoi.PDSfun(Metabolite_pathway_table) call runs without 'unmapped metabolite' errors and produces a PDSmatrix of the expected sample-by-pathway dimensions.
- Spot-check that a known metabolite (e.g., pyruvate) maps to the expected KEGG pathway (e.g., Pyruvate Metabolism) in the resulting table.
- Feature selection (lilikoi.featuresSelection) yields a non-empty set of significant pathways, confirming the mapping produced usable pathway-level features.

## Limitations

- Mapping relies on the identifier vocabularies supported by Lilikoi (metabolite names, HMDB, KEGG, PUBCHEM, ChEBI); metabolites outside these vocabularies will not be converted.
- Name-based matching ('name') can be ambiguous for metabolites with synonyms or inconsistent spellings across platforms, potentially producing incomplete pathway assignments.
- The README example only demonstrates the 'name' identifier type; behavior and output format for other identifier types (HMDBID, KEGGID, etc.) is inferred and not fully documented in the provided README.

## Evidence

- [readme] MetaTOpathway converts metabolite names into HMDB IDs and a metabolite-pathway table: [evidence span withheld — anchor status: external_doc]
- [abstract] Lilikoi v2.0 supports pathway-level downstream analyses: "Lilikoi v2.0 supports data preprocessing, exploratory analysis, pathway visualization, and metabolite pathway regression."
- [intro] Feature mapping is the first of four Lilikoi modules: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
