---
name: lipid-nomenclature-standardisation
description: Use when an annotated m/z feature table contains raw lipid annotation
  strings (e.g. multi-isomer entries in an all_IsomerNames-style column) that must
  be collapsed into standard lipid categories and subclasses before downstream statistics.
when_to_use_negative:
- Annotations are already standardised class-level lipid identifiers, so RefineLipids
  would collapse information you still need
- The refinement goal is pathway-correlation prioritisation of annotations rather
  than nomenclature simplification — use SpaMTP's CalculateAnnotationStatistics /
  CalculateSingleAnnotationStatistics instead
- The annotation strings are non-lipid metabolites that rgoslin's lipid nomenclature
  parser cannot resolve
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaMTP
  role: Provides the RefineLipids function that simplifies lipid nomenclature into
    common lipid categories and classes as a named step of the annotation refinement
    workflow
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: rgoslin
  role: Nomenclature parser used to parse each split lipid name into structured fields
    before category/subclass mapping
- name: R
  role: Execution environment in which the annotation table is loaded, parsed and
    returned
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:51f88a94ebe445537292bf622c03c6449291761cb1ce10dca431fd02827a625b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SpaMTP
  - rgoslin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lipid-nomenclature-standardisation

## Summary

Standardises free-text lipid annotations (e.g. PC(34:1), SM(34:1)) into common lipid categories and classes using SpaMTP's RefineLipids function, which parses lipid names via rgoslin and maps them to the simplified 'lipid_info' level.

## When to use

Use when an annotated m/z feature table contains raw lipid annotation strings (e.g. multi-isomer entries in an all_IsomerNames-style column) that must be collapsed into standard lipid categories and subclasses before downstream statistics.

## When NOT to use

- Annotations are already standardised class-level lipid identifiers, so RefineLipids would collapse information you still need
- The refinement goal is pathway-correlation prioritisation of annotations rather than nomenclature simplification — use SpaMTP's CalculateAnnotationStatistics / CalculateSingleAnnotationStatistics instead
- The annotation strings are non-lipid metabolites that rgoslin's lipid nomenclature parser cannot resolve

## Inputs

- Table of raw lipid annotation strings (e.g. all_IsomerNames-style metadata column from an annotated m/z feature table, such as spotted_Annotated_H_Cl_Adducts.RDS from Zenodo record 17289187)
- R session with SpaMTP and the rgoslin nomenclature parser installed

## Outputs

- Table of standardised lipid identifiers mapped to categories (GL: Glycerolipids, SP: Sphingolipids) and subclasses (DG, TG, PA, PC, PE)
- Per-m/z table keeping the original annotation count alongside the refined count of standardised identifiers

## How to apply

Load the table of raw lipid annotation strings into R (e.g. the all_IsomerNames-style metadata column of an annotated m/z feature table such as the spotted mouse liver dataset on Zenodo record 17289187). Split multi-isomer annotation entries and parse each lipid name into structured fields using rgoslin's nomenclature parser. Map parsed names to standard levels — general categories such as GL (Glycerolipids) and SP (Sphingolipids), and subclasses such as DG, TG, PA, PC, PE — at the simplified 'lipid_info' level described for RefineLipids. Return a table that keeps the original annotation count alongside the refined count of standardised identifiers per m/z, so the collapse of isomers onto shared class-level identifiers is auditable.

## Related tools

- **SpaMTP** (Provides the RefineLipids function that simplifies lipid nomenclature into common lipid categories and classes as a named step of the annotation refinement workflow) — github.com/GenomicsMachineLearning/SpaMTP
- **rgoslin** (Nomenclature parser used to parse each split lipid name into structured fields before category/subclass mapping)
- **R** (Execution environment in which the annotation table is loaded, parsed and returned)

## Examples

```
refined <- SpaMTP::RefineLipids(annotated_peaks)
```

## Evaluation signals

- PC(34:1) and PC(30:2) resolve to the phosphatidylcholine glycerolipid (GL) subclass and PE(38:1) resolves to the phosphatidylethanolamine glycerolipid subclass
- SM(34:1) resolves to the sphingolipid (SP) class
- Refined annotation count per m/z is never greater than the original annotation count, with a maximum reported reduction from 161 to 1
- Output table retains both the original and refined counts per m/z, allowing the isomer-collapse to be audited
- Every input lipid string either maps to a category/subclass at the lipid_info level or is flagged as unparseable by rgoslin

## Limitations

- The reduction magnitude is data-dependent: the paper reports a maximum reduction from 161 to 1, which may not generalise to other datasets
- Parsing depends on rgoslin's ability to resolve the input nomenclature dialect; unresolvable or non-standard lipid strings will not map cleanly
- No specific SpaMTP version number is stated for this workflow, so reproducibility requires recording the installed version

## Evidence
