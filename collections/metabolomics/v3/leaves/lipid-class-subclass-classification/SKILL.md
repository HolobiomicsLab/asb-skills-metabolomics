---
name: lipid-class-subclass-classification
description: Use when an annotated m/z feature table (e.g. an all_IsomerNames-style
  metadata column from a spatial metabolomics experiment) holds multi-isomer lipid
  annotation strings that need standardising into common lipid categories and classes.
when_to_use_negative:
- The input contains non-lipid metabolite annotations — use pathway-based refinement
  via CalculateAnnotationStatistics or CalculateSingleAnnotationStatistics instead
- Annotations are already standardised to common lipid categories/classes, so no nomenclature
  simplification would change the counts
- You need de novo structural annotation from spectra rather than reclassification
  of existing name strings — the pseudo-MS/MS MS1-ID/GNPS route is the appropriate
  step
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpaMTP
  role: Provides the RefineLipids function that simplifies lipid nomenclature into
    common lipid categories and classes as a named step in the annotation refinement
    pipeline
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: rgoslin
  role: Parses each lipid name into structured fields so they can be mapped to standard
    lipid category and subclass levels
- name: R
  role: Language in which the RefineLipids refinement step and surrounding SpaMTP
    pipeline run
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:fc76b2003f87b68374d237d55620b748140255c5b0897ad1b9e185b7a60f381a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - R
  - SpaMTP
  - rgoslin
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lipid-class-subclass-classification

## Summary

SpaMTP's RefineLipids workflow simplifies raw lipid annotation strings from annotated m/z feature tables into common lipid categories and classes (e.g. GL: Glycerolipids, SP: Sphingolipids; subclasses DG, TG, PA, PC, PE) using rgoslin's nomenclature parser, collapsing redundant multi-isomer entries per m/z.

## When to use

Use when an annotated m/z feature table (e.g. an all_IsomerNames-style metadata column from a spatial metabolomics experiment) holds multi-isomer lipid annotation strings that need standardising into common lipid categories and classes.

## When NOT to use

- The input contains non-lipid metabolite annotations — use pathway-based refinement via CalculateAnnotationStatistics or CalculateSingleAnnotationStatistics instead
- Annotations are already standardised to common lipid categories/classes, so no nomenclature simplification would change the counts
- You need de novo structural annotation from spectra rather than reclassification of existing name strings — the pseudo-MS/MS MS1-ID/GNPS route is the appropriate step

## Inputs

- Annotated m/z feature table with raw lipid annotation strings (e.g. all_IsomerNames-style metadata column; spotted_Annotated_H_Cl_Adducts.RDS from Zenodo record 17289187)
- Multi-isomer lipid name entries in common lipid nomenclature (e.g. PC(34:1), PE(38:1), SM(34:1))

## Outputs

- Refined table mapping each m/z to standardised lipid categories and subclasses at the 'lipid_info' level (e.g. GL: Glycerolipids, SP: Sphingolipids; DG, TG, PA, PC, PE)
- Per-m/z comparison of original annotation count vs refined count of standardised identifiers

## How to apply

Load the table of raw lipid annotation strings into R (e.g. the all_IsomerNames-style column from an annotated m/z feature table such as the spotted mouse liver dataset on Zenodo record 17289187). Split multi-isomer entries and parse each lipid name into structured fields with rgoslin's nomenclature parser. Map parsed names to standard category and subclass levels at the simplified 'lipid_info' level — general categories like GL: Glycerolipids and SP: Sphingolipids, and subclasses such as DG, TG, PA, PC, PE — as a named step in SpaMTP's annotation refinement pipeline. Return a table that keeps the original annotation count alongside the refined count of standardised identifiers per m/z so the reduction (paper reports a maximum collapse from 161 to 1) is auditable.

## Related tools

- **SpaMTP** (Provides the RefineLipids function that simplifies lipid nomenclature into common lipid categories and classes as a named step in the annotation refinement pipeline) — https://github.com/GenomicsMachineLearning/SpaMTP
- **rgoslin** (Parses each lipid name into structured fields so they can be mapped to standard lipid category and subclass levels)
- **R** (Language in which the RefineLipids refinement step and surrounding SpaMTP pipeline run) — https://github.com/GenomicsMachineLearning/SpaMTP

## Examples

```
ann <- readRDS("spotted_Annotated_H_Cl_Adducts.RDS"); refined <- RefineLipids(ann)
```

## Evaluation signals

- PC(34:1) and PC(30:2) resolve to the phosphatidylcholine (PC) subclass under the glycerolipid/glycerophospholipid category
- PE(38:1) resolves to the phosphatidylethanolamine (PE) glycerolipid class
- SM(34:1) resolves to the sphingolipid (SP) class
- Annotation count per m/z is reduced, with the paper's maximum reduction from 161 to 1
- Output table retains both the original annotation count and the refined count of standardised identifiers per m/z

## Limitations

- Refinement with paired targeted metabolic data is a placeholder in the vignette ('To come!'), so this route cannot yet incorporate targeted assay evidence
- No explicit SpaMTP version number is given in the article or vignette, so reproducibility depends on recording the installed version yourself
- Parsing quality depends on rgoslin recognising the input nomenclature; unparseable lipid name variants will not map to standard categories/classes

## Evidence

- [readme] SpaMTP is the R framework this skill runs in: [evidence span withheld — anchor status: external_doc]
- [readme] m/z metabolite annotation is the pipeline stage this filter refines: [evidence span withheld — anchor status: external_doc]
