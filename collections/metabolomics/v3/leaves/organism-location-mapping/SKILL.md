---
name: organism-location-mapping
description: Use when a COCONUT subset must be attributed to source organisms and
  their geographic locations — e.g. to compute provenance overlap statistics or continent-level
  distributions from the molecule-organism-location records.
when_to_use_negative:
- The input is a generic chemical structure file (SMILES/SDF) with no organism or
  provenance annotations — structure-level QC or descriptor computation is needed
  instead.
- The analysis goal is chemical classification (ClassyFire/NPclassifier) or annotation-level
  star scoring rather than geographic attribution.
- Only structure-centric properties (Murcko frameworks, Ertl functional groups, deepSMILES)
  are of interest and provenance/location is irrelevant.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3335
tools:
- name: COCONUT
  role: Primary provenance source database whose PostgreSQL pivot tables map molecules
    to organisms and their geographic locations
  repo: https://github.com/mSorok/COCONUT
- name: NaturalProductsOnline
  role: Web application / MongoDB front-end for querying and downloading COCONUT NP
    provenance data
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: MongoDB
  role: Storage backend used to query sourceNaturalProduct and uniqueNaturalProduct
    collections for organism and location annotations
  repo: https://github.com/mSorok/NaturalProductsOnline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:074f4b75c988ca4dd12cb76fc1acc4899db2a0671bffa8c041f9ddcd6b34097f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/COCONUT
  tools:
  - COCONUT
  - MongoDB
  - NaturalProductsOnline
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# organism-location-mapping

## Summary

Maps COCONUT natural products to the organisms that synthesize them and, in turn, to the geographic location where those organisms were collected, enabling continent-level provenance analysis across the 695,133-molecule database. This skill bridges the COCONUT PostgreSQL provenance pivot tables (molecule–organism–location) to geographic aggregation statistics.

## When to use

Use when a COCONUT subset must be attributed to source organisms and their geographic locations — e.g. to compute provenance overlap statistics or continent-level distributions from the molecule-organism-location records.

## When NOT to use

- The input is a generic chemical structure file (SMILES/SDF) with no organism or provenance annotations — structure-level QC or descriptor computation is needed instead.
- The analysis goal is chemical classification (ClassyFire/NPclassifier) or annotation-level star scoring rather than geographic attribution.
- Only structure-centric properties (Murcko frameworks, Ertl functional groups, deepSMILES) are of interest and provenance/location is irrelevant.

## Inputs

- COCONUT PostgreSQL dump (release 2024) containing molecule–organism–location provenance pivot tables
- COCONUT MongoDB dump (COCONUT2020-07 or COCONUT_2021_03) with sourceNaturalProduct and uniqueNaturalProduct collections
- List or table of natural product identifiers with associated organism and geographic-location records

## Outputs

- Continent-level provenance counts (e.g. NPs per continent of organism origin)
- Provenance overlap statistics — sets of NPs shared across continents (Fig. 4-style analysis)
- Figure or table highlighting which continents dominate (Asia), are strong (Africa), sparse (Europe), or absent (Australia) in NP provenance

## How to apply

Start from a COCONUT snapshot (PostgreSQL release 2024 dump, or the MongoDB dump loaded via `mongorestore --db=COCONUT2020-07 --noIndexRestore .` followed by the sourceNaturalProduct/uniqueNaturalProduct index creation documented in the README). Identify the provenance pivot records linking each natural product to its synthesizing organism and the geographic location of that organism; the article notes that COCONUT records 2,653 geographical locations and 55,252 organisms against 695,133 molecules. Filter to molecules whose organism location is known (the article warns that organism_parts and locations in the pivot tables were left nullified for now, so expect substantial missing-location data and treat unknown location as a separate category rather than imputing it). Aggregate counts per continent and compute pairwise provenance overlap statistics — the sets of NPs shared between continents — following the methodology underlying Fig. 4 of the COCONUT paper. Validate by checking that the geographic vocabulary maps cleanly onto continents (Asia, Africa, Europe, etc.) and that the known-location subset size is reported alongside the aggregated counts.

## Related tools

- **COCONUT** (Primary provenance source database whose PostgreSQL pivot tables map molecules to organisms and their geographic locations) — https://github.com/mSorok/COCONUT
- **NaturalProductsOnline** (Web application / MongoDB front-end for querying and downloading COCONUT NP provenance data) — https://github.com/mSorok/NaturalProductsOnline
- **MongoDB** (Storage backend used to query sourceNaturalProduct and uniqueNaturalProduct collections for organism and location annotations) — https://github.com/mSorok/NaturalProductsOnline

## Examples

```
mongorestore --db=COCONUT2020-07 --noIndexRestore . && mongo --port 27022 --eval 'db.sourceNaturalProduct.createIndex({source:1}); db.sourceNaturalProduct.createIndex({simpleInchiKey:1})'
```

## Evaluation signals

- The count of NPs with a known organism location is explicitly reported and is a strict subset of the 695,133 total molecules in COCONUT.
- Aggregate counts per continent sum to the number of NPs with known geographic provenance (no double counting of molecules across continent buckets unless provenance overlap statistics explicitly track shared sets).
- The output reproduces the qualitative trends of the article's Fig. 4: Asia dominant, Africa strong, Europe sparse, Australia absent.
- The geographic vocabulary used matches the 2,653 location entries in COCONUT, normalized to continent level without inventing new location labels.
- Molecules with nullified or missing location fields are excluded (or explicitly labeled as 'unknown') rather than silently imputed.

## Limitations

- Provenance data (organisms and their geographic locations) is a stated final aim of COCONUT not yet fully provided, so location coverage is incomplete.
- The pivot table fields organism_parts and locations were left nullified in the dataset, meaning many molecules will lack a populated location.
- Geographic provenance can be ambiguous when an organism was collected in one place but reported elsewhere; the article does not provide a disambiguation rule.

## Evidence

- [discussion] The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter.: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
- [discussion] nullified organism_parts and locations in the pivot tables for now.: "nullified organism_parts and locations in the pivot tables for now."
- [methods] The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules"
- [readme] mongorestore --db=COCONUT2020-07 --noIndexRestore .: [evidence span withheld — anchor status: not_found]
- [results] COCONUT has NP data from 63 sources (as of sept 2024) ranging from supplimentray resources to well curated databases.: "COCONUT has NP data from 63 sources (as of sept 2024) ranging from supplimentray resources to well curated databases."
