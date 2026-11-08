---
name: inchikey-structure-dereplication
description: Use when preparing a candidate structure database (e.g. DNP, UNPD, GNPS
  libraries) for ISDB spectral matching, where duplicate structures and stereoisomers
  would otherwise inflate candidate ranking lists.
when_to_use_negative:
- The analysis requires distinguishing stereoisomers (e.g. enantiomeric or diastereomeric
  assignment), since the first 14 InChIKey characters collapse them.
- Input entries lack resolvable biological source information down to species level,
  which the paper's downstream taxonomic pipeline requires.
- You are dereplicating at the spectra level rather than the structure level (e.g.
  merging duplicate GNPS library spectra) — this skill operates on structures.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Converts candidate structures to their 27-character InChIKey for first-layer
    dereplication
- name: ISDB-DNP
  role: Downstream in silico spectral database built from the dereplicated structure
    set, backed by the Dictionary of Natural Products
  repo: http://oolonek.github.io/ISDB/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:9a94a416c0ea116f0272ddda4455464639dce2565a843bbb7f880527f0dbb705
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ISDB-DNP
  - MS-Finder
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inchikey-structure-dereplication

## Summary

Dereplicates candidate structures by converting them to InChIKeys and collapsing duplicate structures and stereoisomers using the first connectivity layer (first 14 characters) of the 27-character InChIKey, yielding a non-redundant structure list for spectral matching and taxonomically informed scoring.

## When to use

Use when preparing a candidate structure database (e.g. DNP, UNPD, GNPS libraries) for ISDB spectral matching, where duplicate structures and stereoisomers would otherwise inflate candidate ranking lists.

## When NOT to use

- The analysis requires distinguishing stereoisomers (e.g. enantiomeric or diastereomeric assignment), since the first 14 InChIKey characters collapse them.
- Input entries lack resolvable biological source information down to species level, which the paper's downstream taxonomic pipeline requires.
- You are dereplicating at the spectra level rather than the structure level (e.g. merging duplicate GNPS library spectra) — this skill operates on structures.

## Inputs

- Candidate structure records (e.g. from Dictionary of Natural Products v 27.1, UNPD, KNApSAcK, NPAtlas, FooDB) with structural representations (SMILES/Molfile/InChI)
- MGF spectral files and annotation outputs to which the deduplicated structures feed via ISDB-DNP matching

## Outputs

- Non-redundant structure list keyed by first-layer (14-character) InChIKey
- Dereplicated spectral library/annotation candidate table (e.g. ISDB-DNP prepared library)

## How to apply

Convert each candidate structure to its InChIKey, a 27-character hashed version of the full InChI, using RDKit. Dereplicate by keeping only distinct InChIKeys according to the first layer (the first 14 characters), which collapses duplicates and stereoisomers into a single representative entry. In the paper's database preparation, this step was applied alongside structural filtering — retaining parent masses between 100 and 1,500 Da and spectra with at least 6 fragments — before building the ISDB-DNP spectral library used for annotation. The rationale is that the first InChIKey layer encodes the constitutional skeleton, so compounds differing only in stereochemistry share it and would produce redundant candidates with near-identical spectra during ISDB-DNP matching.

## Related tools

- **RDKit** (Converts candidate structures to their 27-character InChIKey for first-layer dereplication)
- **ISDB-DNP** (Downstream in silico spectral database built from the dereplicated structure set, backed by the Dictionary of Natural Products) — http://oolonek.github.io/ISDB/

## Evaluation signals

- Every retained entry has a 27-character InChIKey, and the set of first 14 characters is unique across the output.
- Count of output entries equals count of distinct first-layer InChIKeys (no duplicate connectivity classes remain).
- All retained structures pass the parent-mass window (100–1,500 Da) applied during the same database preparation.
- Stereoisomeric variants of the same skeleton resolve to a single entry (identical first 14 InChIKey characters).

## Limitations

- First-layer InChIKey dereplication deliberately discards stereochemistry, so stereoisomers cannot be distinguished downstream.
- The benchmarking dataset contained problematic entries (e.g. 197 entries with >0.01 Da mass differences between experimental parent ion mass and calculated exact mass), and erroneous entries cannot be identified by annotation tools.

## Evidence

- [discussion] converting structures to InChIKey: "converting structures to their InChIKey, a 27-character hashed version of the full InChI"
- [discussion] remove duplicates and stereoisomers: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] mass window filter during database preparation: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
