---
name: r-function-implementation
description: Use when a lipid annotation column (e.g. an all_IsomerNames-style metadata
  column with multi-isomer entries) must be simplified into standard lipid nomenclature
  as a filter step in the SpaMTP refinement workflow.
when_to_use_negative:
- The annotation column contains non-lipid metabolites that rgoslin cannot parse into
  lipid nomenclature — the lipid category/class mapping is lipid-specific
- Annotations are already standardised, so the original-vs-refined count comparison
  adds no information
- A different refinement axis is needed, e.g. pathway-correlation prioritisation (use
  CalculateAnnotationStatistics or CalculateSingleAnnotationStatistics instead)
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SpaMTP
  role: Hosts the RefineLipids function used to simplify lipid nomenclature into common
    lipid categories and classes within the annotation refinement pipeline
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: rgoslin
  role: Nomenclature parser that splits multi-isomer entries and parses each lipid
    name into structured fields before mapping to standard categories/subclasses
- name: R
  role: 'Implementation language: the filter is an R function operating on annotation
    metadata tables loaded from .RDS feature tables'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:8f510be5de9860f67689ba858ed7828afb46c0a4f24edfd8e6e1d7ffede3a156
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SpaMTP
  - rgoslin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# r-function-implementation

## Summary

Implement RefineLipids, a named SpaMTP annotation-refinement step in R that parses raw lipid annotation strings with rgoslin and maps them onto common lipid categories and classes (e.g. GL: Glycerolipids, SP: Sphingolipids; DG, TG, PC, PE subclasses).

## When to use

Use when a lipid annotation column (e.g. an all_IsomerNames-style metadata column with multi-isomer entries) must be simplified into standard lipid nomenclature as a filter step in the SpaMTP refinement workflow.

## When NOT to use

- The annotation column contains non-lipid metabolites that rgoslin cannot parse into lipid nomenclature — the lipid category/class mapping is lipid-specific
- Annotations are already standardised, so the original-vs-refined count comparison adds no information
- A different refinement axis is needed, e.g. pathway-correlation prioritisation (use CalculateAnnotationStatistics or CalculateSingleAnnotationStatistics instead)

## Inputs

- Table of raw lipid annotation strings (e.g. all_IsomerNames-style metadata column from an annotated m/z feature table)
- spotted_Annotated_H_Cl_Adducts.RDS from Zenodo record 17289187 (mouse liver dataset with spotted chemical standards)
- SpaMTP package installed from GitHub (devtools::install_github)

## Outputs

- Table of refined lipid annotations at the simplified 'lipid_info' level (categories such as GL: Glycerolipids and SP: Sphingolipids; subclasses DG, TG, PA, PC, PE)
- Per-m/z counts of both original and refined/standardised annotations for validation

## How to apply

Load a table of raw lipid annotation strings into R — e.g. the all_IsomerNames-style metadata column from an annotated m/z feature table such as the spotted mouse liver dataset (Zenodo record 17289187, spotted_Annotated_H_Cl_Adducts.RDS). Split multi-isomer annotation entries and parse each lipid name into structured fields with rgoslin's nomenclature parser. Map parsed names to the simplified 'lipid_info' level of standard lipid categories and subclasses (general categories such as GL: Glycerolipids and SP: Sphingolipids; subclasses such as DG, TG, PA, PC, PE). Return a table that keeps the original annotation count alongside the refined count of standardised identifiers per m/z, so the collapse (the paper reports a maximum reduction from 161 to 1 annotations per m/z) is auditable. The function sits as step 2 in SpaMTP's annotation refinement, after pathway-based prioritisation with CalculateAnnotationStatistics.

## Related tools

- **SpaMTP** (Hosts the RefineLipids function used to simplify lipid nomenclature into common lipid categories and classes within the annotation refinement pipeline) — github.com/GenomicsMachineLearning/SpaMTP
- **rgoslin** (Nomenclature parser that splits multi-isomer entries and parses each lipid name into structured fields before mapping to standard categories/subclasses)
- **R** (Implementation language: the filter is an R function operating on annotation metadata tables loaded from .RDS feature tables)

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- PC(34:1)/PC(30:2) and PE(38:1) resolve to the phosphatidylcholine/phosphatidylethanolamine glycerolipid classes
- SM(34:1) resolves to the sphingolipid class (SP: Sphingolipids)
- Every parsed name maps to one of the standard subclasses (DG, TG, PA, PC, PE) or a top-level category (GL, SP)
- Output table retains both original and refined annotation counts per m/z; per-m/z counts should only stay equal or decrease (paper maximum reduction 161 → 1)
- Multi-isomer entries are split so that each isomer string is parsed individually rather than passed as one name

## Limitations

- No explicit SpaMTP version number is given in the vignette, so the installed version must be recorded by the user
- Parsing depends on rgoslin correctly recognising each lipid name string; unparsable or non-standard names will not map to a standard category/class
- The paired targeted metabolomics refinement section of the vignette is a placeholder only ('To come!'), so RefineLipids cannot yet be cross-validated against targeted data within SpaMTP's documented workflow

## Evidence

- [readme] SpaMTP is an R package with dedicated annotation functions that the new filter function extends: [evidence span withheld — anchor status: external_doc]
