---
name: organism-taxonomy-source-classification
description: Use when a natural-product dataset records a producing organism per molecule
  and you need to assign each NP to a taxonomic category (e.g. plants, bacteria, fungi,
  animals, marine) before computing provenance-level distributions or overlaps.
when_to_use_negative:
- The dataset lacks a populated organism field for the records of interest — COCONUT
  itself notes that provenance annotation is still an outstanding aim and some fields
  (organism_parts, locations) are left null.
- The goal is chemical-structure classification (e.g. ClassyFire chemical classes)
  rather than taxonomic source classification — these are distinct annotation layers.
- Only stereochemistry/unified-structure information is required and no organism-level
  provenance is needed.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Read the COCONUT snapshot, filter to records with a producing organism, and
    map organism identifiers to taxonomic source categories
  repo: https://github.com/pandas-dev/pandas
- name: matplotlib
  role: Render the per-category NP distribution and the superkingdom overlap matrix
    as the figure mirroring Fig. 3
  repo: https://github.com/matplotlib/matplotlib
- name: MongoDB
  role: Underlying store for COCONUT data dumps that can be restored locally for taxonomy-backed
    queries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:5779a47b887ae458c333559497d7e3197cf8d335479c56fc34252ad78faeca98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pandas-dev/pandas
  tools:
  - MongoDB
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# organism-taxonomy-source-classification

## Summary

Classify COCONUT natural products by the taxonomic category of their provenance organism — mapping the organism field on each NP record to the taxonomic ranks used in COCONUT's organism annotation so NPs can be counted and compared across taxonomic sources.

## When to use

Use when a natural-product dataset records a producing organism per molecule and you need to assign each NP to a taxonomic category (e.g. plants, bacteria, fungi, animals, marine) before computing provenance-level distributions or overlaps.

## When NOT to use

- The dataset lacks a populated organism field for the records of interest — COCONUT itself notes that provenance annotation is still an outstanding aim and some fields (organism_parts, locations) are left null.
- The goal is chemical-structure classification (e.g. ClassyFire chemical classes) rather than taxonomic source classification — these are distinct annotation layers.
- Only stereochemistry/unified-structure information is required and no organism-level provenance is needed.

## Inputs

- COCONUT dataset snapshot (PostgreSQL release-2024 export) with per-record producing organism annotation
- Organism taxonomy lookup mapping organism names/ranks/IRIs to the five COCONUT source categories

## Outputs

- Table of NPs tagged with a single taxonomic source category (plants, bacteria, fungi, animals, marine)
- Per-category NP counts and percentage shares of the NPs with a known producer organism
- Pairwise superkingdom overlap matrix of shared NP provenance

## How to apply

Load the COCONUT snapshot (PostgreSQL export from the 2024 release, or the MongoDB dump restored via mongorestore) into a pandas DataFrame. Retain only entries that carry a populated organism field; the article notes that provenance annotation (organisms and their geographic locations) is still a curatorial aim, so missing organism values are expected. Map each organism name/rank/IRI to the five source categories used in the COCONUT taxonomic annotation — plants, bacteria, fungi, animals, marine — joining on the organism identifier rather than free text. For any category whose assignment is ambiguous (e.g. marine-plant), record the mapping rule and propagate it consistently across all NPs. Once classified, count NPs per category and compute pairwise superkingdom overlap matrices (molecules attributed to more than one source kingdom) to quantify shared provenance, as in the Fig. 3 analysis: plants dominate (~65% of NPs with a known producer), animals contribute only ~0.5%, and the largest overlaps are plants–marine and plants–fungi.

## Related tools

- **pandas** (Read the COCONUT snapshot, filter to records with a producing organism, and map organism identifiers to taxonomic source categories) — https://github.com/pandas-dev/pandas
- **matplotlib** (Render the per-category NP distribution and the superkingdom overlap matrix as the figure mirroring Fig. 3) — https://github.com/matplotlib/matplotlib
- **MongoDB** (Underlying store for COCONUT data dumps that can be restored locally for taxonomy-backed queries)

## Examples

```
df = pd.read_csv('coconut_2024.csv'); df = df[df['organism'].notna()]; df['source'] = df['organism'].map(TAX_MAP); df.groupby('source')['compound_id'].count()
```

## Evaluation signals

- Percentage of NPs falling into the plant-only category should reproduce the reported ~65% (within rounding tolerance).
- Animal-origin NPs should account for roughly 0.5% of the NPs with a known producing organism.
- Every NP with a non-null organism field must receive exactly one of the five source-category labels; no NP should be left unlabelled or double-labelled.
- The recomputed pairwise superkingdom overlap matrix should show plants–marine and plants–fungi as the two largest cross-category overlaps.
- The count of NPs used in the analysis must equal the number of COCONUT records with a populated organism field (i.e. filtering step is auditable).

## Limitations

- Provenance data — organisms synthesizing NPs and geographic location — is described as a final aim not yet fully provided, so some records will lack organism annotation and must be excluded.
- The organism_parts and locations pivot-table fields are nullified in the current release, so finer-grained organism-part or location breakdowns are unavailable.
- Ambiguous or multi-kingdom organism attributions require a documented mapping rule; otherwise category assignment may be inconsistent across the dataset.
- Only records from the COCONUT 2024 release (PostgreSQL export) or the equivalent MongoDB dump were validated; other snapshots may have different organism annotation conventions.

## Evidence

- [discussion] Provenance data (organisms synthesizing NPs and geographic location of organisms) is a final aim not yet fully provided: "The final aim of COCONUT is to provide to the scientific community NPs structures and their provenance, i.e.. organisms that synthesize them and geographic location of the latter."
- [discussion] Pivot table fields organism_parts and locations were nullified: "nullified organism_parts and locations in the pivot tables for now."
- [readme] Source organisms are part of the COCONUT annotation set and are linked per molecule: [evidence span withheld — anchor status: not_found]
- [readme] COCONUT 2.0 search and filtering includes filtering by organism: [evidence span withheld — anchor status: external_doc]
