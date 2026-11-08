---
name: smiles-scaffold-building-block-annotation
description: Use when building an in silico structure database of plausible conjugated
  metabolites (e.g. bile acid conjugates) from a defined scaffold set and building-block
  SMILES, to enable COSMIC/SIRIUS-based structural annotation.
when_to_use_negative:
- The target structures are already present in an existing structure database (e.g.
  the HMDB structure database, 113,983 compounds) — enumeration is for metabolites
  absent from spectral/structure libraries
- The question is structure annotation or confidence scoring from MS/MS spectra rather
  than generating candidate structures — use COSMIC/SIRIUS 4 with CSI:FingerID instead
- Stereochemistry must be resolved — only two-dimensional structures are considered,
  as stereochemistry cannot be established from fragmentation spectra
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3336
tools:
- name: SmiLib v2.0
  role: 'Combinatorial enumeration engine: attaches amino-acid/acyl building blocks
    at scaffold conjugation and phase-2 modification sites to generate the conjugate
    structures'
- name: COSMIC
  role: Consumes the enumerated 28,630-structure database as the in silico structure
    database for high-confidence annotation
- name: SIRIUS 4
  role: Processes LC-MS/MS runs against the generated structure database downstream
    (formula determination, CSI:FingerID ranking)
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:19fbbb834c2f0ee0aaa32ad753cdabdc9e41b644d5ae44ab6e9970726e13c56f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  - SmiLib v2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# smiles-scaffold-building-block-annotation

## Summary

Exhaustively enumerates combinatorial conjugate structures by attaching amino-acid/acyl building-block SMILES onto curated molecular scaffolds (e.g. bile acid cores with phase-2 modifications) using SmiLib v2.0, producing the in silico search database that COSMIC then annotates.

## When to use

Use when building an in silico structure database of plausible conjugated metabolites (e.g. bile acid conjugates) from a defined scaffold set and building-block SMILES, to enable COSMIC/SIRIUS-based structural annotation.

## When NOT to use

- The target structures are already present in an existing structure database (e.g. the HMDB structure database, 113,983 compounds) — enumeration is for metabolites absent from spectral/structure libraries
- The question is structure annotation or confidence scoring from MS/MS spectra rather than generating candidate structures — use COSMIC/SIRIUS 4 with CSI:FingerID instead
- Stereochemistry must be resolved — only two-dimensional structures are considered, as stereochemistry cannot be established from fragmentation spectra

## Inputs

- Curated scaffold SMILES set (322 bile acid scaffolds with phase-2 modifications)
- Building-block SMILES list (91 amino-acid/acyl fragments)
- SmiLib v2.0 scaffold and building-block input files

## Outputs

- bile_acid_conjugates.smi — canonical SMILES, one structure per line (28,630 structures)
- Combinatorial structure database for COSMIC in silico annotation

## How to apply

Load a curated scaffold set (here, 322 bile acid scaffolds spanning cholic acid, deoxycholic acid, hyocholic acid and chenodeoxycholic acid cores with phase-2 modifications) and 91 amino-acid/acyl building-block SMILES into SmiLib v2.0 scaffold and building-block input files. Configure the enumeration so fragments attach at each scaffold's conjugation positions and phase-2 modification sites, then run exhaustive combinatorial enumeration to generate all scaffold-building-block conjugates. Count the enumerated records — the COSMIC bile acid search database required exactly 28,630 structures — and reject the run if the count differs. Write the enumerated conjugates to a .smi file of canonical SMILES, one structure per line; this file is the structure database consumed by COSMIC's in silico annotation workflow alongside SIRIUS 4 and CSI:FingerID candidate ranking.

## Related tools

- **SmiLib v2.0** (Combinatorial enumeration engine: attaches amino-acid/acyl building blocks at scaffold conjugation and phase-2 modification sites to generate the conjugate structures)
- **COSMIC** (Consumes the enumerated 28,630-structure database as the in silico structure database for high-confidence annotation)
- **SIRIUS 4** (Processes LC-MS/MS runs against the generated structure database downstream (formula determination, CSI:FingerID ranking)) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Output file bile_acid_conjugates.smi exists and its row count equals exactly 28,630 structures
- Every enumerated record is a valid canonical SMILES, one per line
- Each conjugate contains a bile acid core scaffold with at least one attached amino-acid/acyl building block at a conjugation or phase-2 modification position
- Database size matches the search database size used for COSMIC's in silico structure database generation
- Downstream: >90% correctness for top-12 COSMIC-suggested bile acid conjugates from this database is a reference-level quality indicator

## Limitations

- Only two-dimensional structures are considered; stereochemistry cannot be established from fragmentation spectra
- Small structural modifications can be hard or impossible to tell apart using MS/MS data alone — enumerated near-duplicates may not be distinguishable by annotation
- Enumeration is exhaustive and combinatorial; scaffold/building-block choices (322 scaffolds, 91 blocks) define the plausible chemistry space and will miss conjugates outside that set

## Evidence

- [abstract] COSMIC workflow combining in silico structure database generation with annotation: "Here we introduce the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score"
- [discussion] >90% correctness of top-12 bile acid conjugate annotations from the database: "more than 90% of the annotations for the top 12 bile acid conjugates suggested by COSMIC turned out to be correct."
- [discussion] Stereochemistry not considered — 2D structures only: "hence, only the two-dimensional structure is considered"
- [readme] SIRIUS framework is the downstream analysis engine consuming LC-MS/MS data against generated databases: [evidence span withheld — anchor status: external_doc]
