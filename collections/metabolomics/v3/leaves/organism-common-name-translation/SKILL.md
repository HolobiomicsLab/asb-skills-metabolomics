---
name: organism-common-name-translation
description: Use when a natural-products occurrence table contains free-text organism
  names — vernacular names, TCM names, or misspelled binomials — that must be mapped
  to standardized taxa before aggregation.
when_to_use_negative:
- Organism strings are already canonical scientific binomials with no vernacular or
  TCM names present — skip translation and go directly to taxonomy resolution
- Names shorter than four characters (e.g. 'Aye') — these common names are removed
  by the pipeline and cannot be reliably translated
- You need a mapping to a specific lineage database like NCBI taxonomy rather than
  the Open Tree of Life
tools:
- name: R
  role: Host language for the 4_cleaningTaxonomy.R and 5_addingOTL.R pipeline stages
  repo: https://github.com/lotusnprod/lotus-processor
- name: stringdist
  role: Approximate string matching to translate common names to scientific names
- name: rotl
  role: R interface to the Open Tree of Life used to resolve accepted scientific names
    to taxonomy nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:264a10c0b2c61deb85578e501881cfef639d2dde13218f2942b64a699fa8a8f0
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - R
  - rotl
  - stringdist
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# organism-common-name-translation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Translates heterogeneous vernacular/common organism names into canonical scientific names using approximate string matching (stringdist), then resolves them to a unified taxonomy via rotl/Open Tree of Life, storing the mappings as an SQLite dictionary. This is the organism-harmonization step of the LOTUS curation pipeline.

## When to use

Use when a natural-products occurrence table contains free-text organism names — vernacular names, TCM names, or misspelled binomials — that must be mapped to standardized taxa before aggregation.

## When NOT to use

- Organism strings are already canonical scientific binomials with no vernacular or TCM names present — skip translation and go directly to taxonomy resolution
- Names shorter than four characters (e.g. 'Aye') — these common names are removed by the pipeline and cannot be reliably translated
- You need a mapping to a specific lineage database like NCBI taxonomy rather than the Open Tree of Life

## Inputs

- Cleaned organism table: interim/tables/2_cleaned/organism/cleaned.tsv.gz (from 4_cleaningTaxonomy.R)
- Original organism name strings (scientific, vernacular, and TCM names from source NP databases)

## Outputs

- SQLite organism dictionary at interim/dictionaries/organism/otl.sqlite mapping organism names to Open Tree of Life taxonomy nodes

## How to apply

Load the cleaned organism table (interim/tables/2_cleaned/organism/cleaned.tsv.gz) from the 4_cleaningTaxonomy.R stage and first strip organ names of the producing organism to prevent mistranslation, then drop common names of three characters or fewer. Translate remaining common names to scientific names with stringdist approximate matching, discarding pairs where a specific epithet was added (e.g. 'Aloe' → 'Aloe vera') or the name is generic-only, and accept a translation only when both genus and species name agreement exceed 50%. Submit the accepted scientific names to rotl, the R interface to the Open Tree of Life, via the 5_addingOTL.R script to obtain taxonomy nodes. Persist the harmonized organism-name-to-taxon mappings in an SQLite dictionary at interim/dictionaries/organism/otl.sqlite.

## Related tools

- **R** (Host language for the 4_cleaningTaxonomy.R and 5_addingOTL.R pipeline stages) — https://github.com/lotusnprod/lotus-processor
- **stringdist** (Approximate string matching to translate common names to scientific names)
- **rotl** (R interface to the Open Tree of Life used to resolve accepted scientific names to taxonomy nodes)

## Examples

```
Rscript 5_addingOTL.R  # resolve cleaned organism names via rotl and write interim/dictionaries/organism/otl.sqlite
```

## Evaluation signals

- otl.sqlite is generated and populated with mappings for all cleaned organism names
- No translation pair retains a case where a specific epithet was added to a generic-only common name (e.g. 'Aloe' → 'Aloe vera' must be absent)
- All accepted translations satisfy the >50% genus and species consistency threshold
- No common names of three or fewer characters appear in the output dictionary
- Mapping plausibility spot-checked by expert review of a sample of name-to-taxon pairs

## Limitations

- Correct salts/charged/dimeric analog of the problem: the pipeline still removes some correct entries and induces or misses some incorrect ones; no exact counts are given
- No quantified accuracy is reported for the automated translation step; it relies on predefined rules that do not cover all cases and can commonly lead to incorrect translations
- The paper describes the mechanism but does not independently validate it; plausibility rests on expert review

## Evidence

- [methods] Organism harmonization proceeds by first resolving original organism strings to canonical scientific names: [evidence span withheld — anchor status: not_found]
- [methods] stringdist approximate matching: [evidence span withheld — anchor status: not_found]
- [methods] rotl OTL resolution: [evidence span withheld — anchor status: not_found]
