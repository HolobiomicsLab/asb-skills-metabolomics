---
name: structure-database-size-validation
description: Use when enumerating a combinatorial search database (e.g. amino-acid
  conjugates of bile acid scaffolds) and you must confirm the enumerated structure
  count matches the expected database size before downstream annotation.
when_to_use_negative:
- When a custom/unrestricted structure search database is desired rather than the
  fixed combinatorial bile acid conjugate space used for COSMIC — the 28,630-count
  validation would not apply.
- When structures already exist in a curated database (e.g. the HMDB structure database
  with 113,983 compounds); no combinatorial enumeration is needed.
edam_operation: http://edamontology.org/operation_2485
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: SmiLib v2.0
  role: Exhaustive combinatorial enumeration of scaffold–building-block conjugate
    structures attached at conjugation and phase-2 modification sites
- name: COSMIC
  role: Downstream workflow for which the validated 28,630-structure in silico search
    database is generated
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes the LC–MS/MS runs searched against the generated structure database
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:f523d768697c2be67a9fa5d786f2748c696fbbb3283b31bc81eac88d6dbc7195
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  - SmiLib v2.0
  edam_operation: http://edamontology.org/operation_2485
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# structure-database-size-validation

## Summary

Validates an in silico combinatorial structure database generation run for COSMIC by exhaustively enumerating bile acid conjugates with SmiLib v2.0 and checking that the output contains exactly 28,630 structures.

## When to use

Use when enumerating a combinatorial search database (e.g. amino-acid conjugates of bile acid scaffolds) and you must confirm the enumerated structure count matches the expected database size before downstream annotation.

## When NOT to use

- When a custom/unrestricted structure search database is desired rather than the fixed combinatorial bile acid conjugate space used for COSMIC — the 28,630-count validation would not apply.
- When structures already exist in a curated database (e.g. the HMDB structure database with 113,983 compounds); no combinatorial enumeration is needed.

## Inputs

- Curated bile acid scaffold set (322 scaffolds; cholic acid, deoxycholic acid, hyocholic acid, chenodeoxycholic acid cores with phase-2 modifications) as SmiLib v2.0 scaffold input file
- 91 amino-acid/acyl building-block SMILES as SmiLib v2.0 building-block input file

## Outputs

- bile_acid_conjugates.smi with 28,630 enumerated conjugate structures as canonical SMILES, one structure per line
- Structure count report confirming exactly 28,630 enumerated records

## How to apply

Load the curated bile acid scaffold set (322 scaffolds spanning cholic acid, deoxycholic acid, hyocholic acid and chenodeoxycholic acid cores with phase-2 modifications) and the 91 amino-acid/acyl building-block SMILES into SmiLib v2.0 scaffold and building-block input files. Configure combinatorial enumeration so the amino-acid/acyl fragments attach at the conjugation positions of each scaffold core and at the phase-2 modification sites. Run the exhaustive enumeration to generate all scaffold–building-block conjugates, count the enumerated records, and reject the run if the count differs from exactly 28,630 — the database size used for COSMIC's in silico structure database generation. On acceptance, write the enumerated conjugates as canonical SMILES to bile_acid_conjugates.smi, one structure per line.

## Related tools

- **SmiLib v2.0** (Exhaustive combinatorial enumeration of scaffold–building-block conjugate structures attached at conjugation and phase-2 modification sites)
- **COSMIC** (Downstream workflow for which the validated 28,630-structure in silico search database is generated) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes the LC–MS/MS runs searched against the generated structure database) — https://github.com/sirius-ms/sirius

## Examples

```
smilib --scaffolds bile_acid_scaffolds.smi --building-blocks amino_acids.smi --output bile_acid_conjugates.smi && wc -l bile_acid_conjugates.smi  # must equal 28630
```

## Evaluation signals

- bile_acid_conjugates.smi exists and its row count equals exactly 28,630 structures
- Every line of the .smi file parses as a valid canonical SMILES
- Every enumerated structure contains a bile acid core scaffold conjugated to at least one amino-acid/acyl building block
- Run is rejected if the enumerated count differs from 28,630, consistent with the database size used for COSMIC's in silico structure database generation
- No duplicate SMILES records in the output file

## Limitations

- Establishing stereochemistry from fragmentation spectra is beyond automated search engines, so only the two-dimensional structure is considered in the generated database.
- The confidence-score threshold (0.64, roughly FDR 10%) is practical guidance only and is no guarantee in either direction, so database annotations downstream cannot be assumed to be error-free.
- Small structure modifications can be hard or impossible to tell apart using MS/MS data alone, an intrinsic limitation of small molecule MS/MS that enumeration of a larger database cannot overcome.

## Evidence

- [results] combinatorially adding amino acids to bile acid cores yielded 28,630 plausible bile acid conjugate structures in the search database: [evidence span withheld — anchor status: not_found]
- [methods] the enumeration is accepted only if the output file exists and its structure row count equals exactly 28,630: [evidence span withheld — anchor status: not_found]
- [methods] 322 scaffolds spanning cholic acid, deoxycholic acid, hyocholic acid and chenodeoxycholic acid cores with phase-2 modifications: [evidence span withheld — anchor status: not_found]
- [abstract] the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score: "Here we introduce the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score"
- [discussion] Establishing the stereochemistry from fragmentation spectra is highly challenging and beyond the power of automated search engines; hence, only the two-dimensional structure is considered: "Establishing the stereochemistry from fragmentation spectra is highly challenging and beyond the power of automated search engines; hence, only the two-dimensional structure is considered"
