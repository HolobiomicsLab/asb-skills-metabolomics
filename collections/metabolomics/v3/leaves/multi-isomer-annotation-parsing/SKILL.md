---
name: multi-isomer-annotation-parsing
description: Use when an annotated spatial metabolomics m/z table contains multiple
  isomer-level lipid annotation strings per peak (e.g. PC(34:1)/PC(30:2)) that need
  collapsing into standard lipid classes (DG, TG, PA, PC, PE, SM) before downstream
  statistics.
when_to_use_negative:
- When the annotation data are not lipid names (RefineLipids specialises in lipid
  nomenclature simplification, not small-molecule or peptide annotation).
- When annotations are already collapsed to a single standardised identifier per m/z
  — re-running adds parsing overhead without reducing multiplicity.
- When lipid names do not conform to a nomenclature rgoslin can parse (e.g. free-text
  or vendor-specific labels).
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: SpaMTP
  role: Provides the RefineLipids function that performs lipid nomenclature simplification
    on annotated m/z features
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: rgoslin
  role: Used within RefineLipids to parse lipid names into structured nomenclature
    fields (category, class, chain information)
- name: Cardinal
  role: Supplies the processed spatial metabolomics data object whose annotated m/z
    feature table RefineLipids operates on
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:9c96946dca80ca064040c0b50854e80e028c5bf71fe248cf1cd8bec6513c28ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - R
  - SpaMTP
  - rgoslin
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multi-isomer-annotation-parsing

## Summary

Parses and simplifies multi-isomer lipid metabolite annotations from spatial metabolomics m/z feature tables into common lipid nomenclature using SpaMTP's RefineLipids function, which converts lipid names into standardised lipid categories and classes via rgoslin parsing.

## When to use

Use when an annotated spatial metabolomics m/z table contains multiple isomer-level lipid annotation strings per peak (e.g. PC(34:1)/PC(30:2)) that need collapsing into standard lipid classes (DG, TG, PA, PC, PE, SM) before downstream statistics.

## When NOT to use

- When the annotation data are not lipid names (RefineLipids specialises in lipid nomenclature simplification, not small-molecule or peptide annotation).
- When annotations are already collapsed to a single standardised identifier per m/z — re-running adds parsing overhead without reducing multiplicity.
- When lipid names do not conform to a nomenclature rgoslin can parse (e.g. free-text or vendor-specific labels).

## Inputs

- Annotated m/z feature table from SpaMTP/Cardinal (e.g. spotted_Annotated_H_Cl_Adducts.RDS from Zenodo record 17289187), with a multi-isomer lipid annotation metadata column
- Lipid annotation strings in standard nomenclature parseable by rgoslin

## Outputs

- Refined annotation table mapping each m/z to simplified lipid categories and classes, retaining original vs refined annotation counts per m/z

## How to apply

Load the annotated m/z feature table (e.g. the spotted mouse liver dataset's all_IsomerNames-style metadata column) into R with SpaMTP, then run RefineLipids. The function splits multi-isomer annotation entries and parses each lipid name into structured fields using rgoslin's nomenclature parser, then maps parsed names to standard lipid category levels (e.g. GL: Glycerolipids, SP: Sphingolipids) and subclasses (e.g. DG, TG, PA, PC, PE, SM). The output retains both the original annotation count and the refined count of standardised identifiers per m/z so the collapse is auditable. Decide on the desired refinement level ('lipid_info' describes the simplified category/subclass level RefineLipids operates at). Run the paper's reference lipids as a validation check.

## Related tools

- **SpaMTP** (Provides the RefineLipids function that performs lipid nomenclature simplification on annotated m/z features) — https://github.com/GenomicsMachineLearning/SpaMTP
- **rgoslin** (Used within RefineLipids to parse lipid names into structured nomenclature fields (category, class, chain information))
- **Cardinal** (Supplies the processed spatial metabolomics data object whose annotated m/z feature table RefineLipids operates on)

## Examples

```
refined <- RefineLipids(annotated_mz_table, annotation_column = "all_IsomerNames")
```

## Evaluation signals

- Check that PC(34:1)/PC(30:2) resolves to the phosphatidylcholine (PC) glycerolipid class.
- Check that PE(38:1) resolves to the phosphatidylethanolamine (PE) glycerolipid class.
- Check that SM(34:1) resolves to the sphingolipid (SP) class.
- Verify the refined annotation count per m/z is less than or equal to the original count (the paper reports a maximum reduction from 161 to 1).
- Confirm every output identifier belongs to one of the standard lipid categories (e.g. GL, SP) or subclasses (e.g. DG, TG, PA, PC, PE).

## Limitations

- The article notes a paired targeted metabolomics comparison section is 'To come!', so integration of RefineLipids with targeted comparison data is not yet documented.
- No specific version of SpaMTP is stated in the annotation-refinement section; record the installed version when reproducing.
- Refinement depends on lipid strings conforming to a nomenclature rgoslin can parse; entries that fail parsing may not be simplified.

## Evidence

- [methods] RefineLipids simplifies lipid annotations by converting lipid names into common lipid categories and classes: [evidence span withheld — anchor status: not_found]
- [methods] general categories such as GL: Glycerolipids and SP: Sphingolipids; subclasses such as DG, TG, PA, PC, PE: [evidence span withheld — anchor status: not_found]
- [results] annotation count per m/z reduced (the paper reports a maximum reduction from 161 to 1): [evidence span withheld — anchor status: not_found]
- [methods] rgoslin's nomenclature parser is used to split multi-isomer annotation entries and parse each lipid name: [evidence span withheld — anchor status: not_found]
