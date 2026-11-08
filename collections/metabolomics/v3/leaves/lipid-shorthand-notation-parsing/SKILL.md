---
name: lipid-shorthand-notation-parsing
description: Use when an uploaded lipidomic matrix or lipid-name list contains vendor
  shorthand (e.g., PE(P-34:1), sphingoid bases like d16:1, placeholders like 0:0)
  that must be normalized before LipidOne 2.0 analysis.
when_to_use_negative:
- The names are already in LipidOne 2.0-compatible molecular-species or sum-composition
  form, so no translation is needed
- Input is a non-lipid metabolomic dataset (e.g., polar metabolites for Metabo Analyst),
  outside lipid shorthand nomenclature
- The analysis targets building-block (LBB) chain-level aggregation rather than lipid
  class (LCL) aggregation
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: LipidOne 2.0
  role: Target platform whose back-end R scripts define the compatible molecular-species
    and sum-composition naming conventions; source of example datasets (ERYTHROCYTES,
    FUNGUS, SCIATIC NERVE)
  repo: https://lipidone.eu
- name: R
  role: Implementation language for the notation parser and class-prefix extraction/aggregation
    logic, mirroring LipidOne 2.0's back-end scripts
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:b6b966bda827504765485c6766803bec193e6531b7907cad3663908783d42dcf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lipid-shorthand-notation-parsing

## Summary

A skill for translating vendor lipid shorthand notations (e.g., PE(P-16:0/18:1)) into LipidOne 2.0-compatible forms and parsing lipid class prefixes from LIPID MAPS shorthand to enable class-level (LCL) aggregation of lipidomic matrices in R.

## When to use

Use when an uploaded lipidomic matrix or lipid-name list contains vendor shorthand (e.g., PE(P-34:1), sphingoid bases like d16:1, placeholders like 0:0) that must be normalized before LipidOne 2.0 analysis.

## When NOT to use

- The names are already in LipidOne 2.0-compatible molecular-species or sum-composition form, so no translation is needed
- Input is a non-lipid metabolomic dataset (e.g., polar metabolites for Metabo Analyst), outside lipid shorthand nomenclature
- The analysis targets building-block (LBB) chain-level aggregation rather than lipid class (LCL) aggregation

## Inputs

- Vendor lipid shorthand names from mass-spectrometric experiments (e.g., PE(P-16:0/18:1), PE(P-34:1))
- Molecular-species-level lipidomic data matrix (CSV) with LIPID MAPS shorthand row names, e.g., LipidOne example dataset ERYTHROCYTES
- Mapping tables (article Table 1 and Table 2) of vendor notation to LipidOne 2.0 target forms

## Outputs

- lipid_name_translations.csv: input names with molecular-species and sum-composition forms plus per-example match status
- lcl_submatrix.csv: lipid class (LCL) sub-matrix with one row per class and the original sample columns

## How to apply

Encode the article's Table 1 and Table 2 vendor-to-LipidOne 2.0 mappings as test vectors, then in R tokenize each name into its headgroup/lipid class, chain descriptors (acyl, alkyl, or alkenyl chains and sphingoid bases such as d16:1), and placeholder positions (e.g., 0:0). Emit both the molecular-species form (chain identities retained, e.g., PE P-16:0_18:1) and the sum-composition form (class-level shorthand, e.g., PE O-34:2). For sub-matrix generation, parse the LIPID MAPS class prefix (PC, PE, TG) from each row name of a molecular-species matrix such as ERYTHROCYTES, group rows by class, and sum abundances per sample column. Write results to lipid_name_translations.csv or lcl_submatrix.csv and validate that every emitted string matches its expected mapping.

## Related tools

- **LipidOne 2.0** (Target platform whose back-end R scripts define the compatible molecular-species and sum-composition naming conventions; source of example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE)) — https://lipidone.eu
- **R** (Implementation language for the notation parser and class-prefix extraction/aggregation logic, mirroring LipidOne 2.0's back-end scripts) — github.com/matteogiulietti/LipidOne

## Examples

```
results <- translate_names(vendor_names); write.csv(results, "lipid_name_translations.csv", row.names = FALSE)
```

## Evaluation signals

- 100% of Table 1 and Table 2 mapping examples reproduce the exact expected molecular-species/sum-composition strings; any mismatch is reported as a failed case
- lcl_submatrix.csv contains exactly one row per distinct lipid class present in the input matrix
- Each aggregated class value in the LCL sub-matrix equals the sum of its constituent molecular-species values for every sample
- lcl_submatrix.csv parses as a well-formed matrix with the same sample columns as the input
- Placeholders (e.g., 0:0) and sphingoid bases (e.g., d16:1) are tokenized without being misread as ordinary acyl chains

## Limitations

- Validation depends entirely on the completeness of the Table 1/Table 2 mapping examples encoded as test vectors; unmapped vendor variants may fail
- LipidOne 2.0 is a web platform (no stated license, Docker/conda install, or persistent dataset repository accession), so example datasets must be obtained via the web UI
- The GitHub README documents the earlier desktop Lipid One, written in Perl for Windows with Gnuplot dependencies, not the R back-end of LipidOne 2.0

## Evidence

- [abstract] LipidOne 2.0 compatible forms: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [abstract] Support Protocol scope: "Support Protocol: Lipid nomenclature from spectrometric experiments"
- [discussion] LIPID MAPS shorthand basis: "Update on LIPID MAPS classification, nomenclature, and short- hand notation for MS-derived lipid structures"
- [intro] LIPID MAPS scale: "According to the Lipid Maps database, more than 48,400 unique lipid structures"
- [readme] Desktop Lipid One implementation caveat: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."
