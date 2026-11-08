---
name: approximate-string-matching
description: Use when free-text organism fields (vernacular, TCM, misspelled scientific
  names) in occurrence tables must be mapped to canonical taxa before aggregation
  or cross-database comparison.
when_to_use_negative:
- Organism names are already resolved to canonical scientific names or Open Tree of
  Life identifiers and only need joining
- The task is chemical structure-to-name or name-to-structure translation — LOTUS
  handles those with separate rule-based steps, not stringdist matching
- Matching reference titles (e.g. against DOI/PMID records with a Damerau-Levenshtein
  distance < 10) — that is reference filtering, not organism harmonization
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: stringdist
  role: R package used to compute approximate string matches between common organism
    names and candidate scientific names
- name: rotl
  role: R interface to the Open Tree of Life used to resolve accepted scientific names
    to unified taxonomy nodes in the 5_addingOTL.R script
- name: R
  role: Implementation language for the organism cleaning and OTL mapping stages (4_cleaningTaxonomy.R,
    5_addingOTL.R) in the lotus-processor pipeline
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:f36cb7d56b68a3d4b5e7d246295e56e0e2a46b6ec0a8af3288a34d044ce4f261
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - R
  - rotl
  - stringdist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# approximate-string-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous organism name strings from natural products source databases into standardized taxon names by translating common names to scientific names with approximate string matching (stringdist in R) and then resolving accepted names to Open Tree of Life taxonomy nodes via rotl.

## When to use

Use when free-text organism fields (vernacular, TCM, misspelled scientific names) in occurrence tables must be mapped to canonical taxa before aggregation or cross-database comparison.

## When NOT to use

- Organism names are already resolved to canonical scientific names or Open Tree of Life identifiers and only need joining
- The task is chemical structure-to-name or name-to-structure translation — LOTUS handles those with separate rule-based steps, not stringdist matching
- Matching reference titles (e.g. against DOI/PMID records with a Damerau-Levenshtein distance < 10) — that is reference filtering, not organism harmonization

## Inputs

- cleaned organism table TSV (interim/tables/2_cleaned/organism/cleaned.tsv.gz)
- original organism name strings from source NP databases (scientific, vernacular, TCM names)
- aggregated common-name-to-scientific-name dictionaries

## Outputs

- SQLite organism-name-to-taxon dictionary (interim/dictionaries/organism/otl.sqlite)
- harmonized taxon identifiers from the Open Tree of Life

## How to apply

Load the cleaned organism table (interim/tables/2_cleaned/organism/cleaned.tsv.gz) produced by 4_cleaningTaxonomy.R. First remove organ names of the producing organism to avoid mistranslation, then drop common names with three or fewer characters. Apply approximate string matching via the stringdist R package to translate common names to scientific names, discarding translation pairs where a specific epithet was added (e.g. 'Aloe' → 'Aloe vera') or where the match is generic-only; accept a translation only when both the species and genus names are consistent at more than 50%. Feed the accepted scientific names to rotl, the R interface to the Open Tree of Life, using the 5_addingOTL.R script, and store the resulting name-to-taxon mappings as an SQLite dictionary at interim/dictionaries/organism/otl.sqlite.

## Related tools

- **stringdist** (R package used to compute approximate string matches between common organism names and candidate scientific names)
- **rotl** (R interface to the Open Tree of Life used to resolve accepted scientific names to unified taxonomy nodes in the 5_addingOTL.R script)
- **R** (Implementation language for the organism cleaning and OTL mapping stages (4_cleaningTaxonomy.R, 5_addingOTL.R) in the lotus-processor pipeline) — https://github.com/lotusnprod/lotus-processor

## Examples

```
Rscript 5_addingOTL.R
```

## Evaluation signals

- interim/dictionaries/organism/otl.sqlite is generated and populated with mappings for all cleaned organism names
- Every accepted translation has genus and species name consistency above 50%
- No accepted translation pair adds a specific epithet (e.g. 'Aloe' → 'Aloe vera') or is generic-only
- No common names with three or fewer characters remain in the translation input
- Mapping plausibility confirmed by expert review of sampled name-to-taxon pairs

## Limitations

- The LOTUS pipeline 'still removes a lot of correct entries and misses or induces some incorrect ones', so aggressive filtering of translation pairs can discard valid mappings
- The paper describes the mechanism but does not independently validate it; mapping quality rests on expert review
- Rule-based matching cannot cover all organism-name variants (vernacular, TCM, abbreviations), and short common names are dropped rather than translated

## Evidence

- [results] only common names with more than three characters were kept: "only common names with more than three characters were kept"
- [results] translation pair discarded (epithet added): "the translation pair was discarded (for example, “Aloe” translated in “Aloe vera” was discarded)"
- [results] genus/species consistency threshold: "If both the species and genus names were consistent at more than 50%, they were considered consistent overall"
- [results] organ names removed before translation: "Organ names of the producing organism were removed to avoid wrong translation"
- [readme] lotus-processor runtime environment: "If you are in a hurry and just want a quick test without reading the whole documentation, what you need is:"
