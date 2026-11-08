---
name: taxonomy-resolution-open-tree-of-life
description: Use when a NP occurrence pipeline contains raw organism name strings
  (vernacular, TCM, common names, organ names) that must be mapped to canonical taxa
  before structure-organism pair harmonization.
when_to_use_negative:
- The organism table already contains canonical scientific names directly resolvable
  without approximate string matching
- Only a small number of known organism names must be matched manually rather than
  via automated translation
- The paper describes this mechanism but does not independently validate it; do not
  rely on it without expert review of mapping plausibility
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: stringdist
  role: Performs approximate string matching between common/vernacular names and candidate
    scientific names, with genus/species consistency thresholds at >50%
- name: rotl
  role: R interface to the Open Tree of Life; resolves accepted scientific names to
    taxonomy nodes for the mapping table
- name: R
  role: Language in which the 4_cleaningTaxonomy.R and 5_addingOTL.R harmonization
    scripts are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:b4248465ad085c8a7b35725495b970301ec1ea393dd37bf925ae635f5a9cd2ff
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - R
  - rotl
  - stringdist
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# taxonomy-resolution-open-tree-of-life

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous organism name strings into a unified taxonomy by first translating common/vernacular/TCM names to scientific names via approximate string matching, then resolving accepted scientific names to Open Tree of Life taxonomy nodes with rotl to build a name-to-taxon SQLite dictionary.

## When to use

Use when a NP occurrence pipeline contains raw organism name strings (vernacular, TCM, common names, organ names) that must be mapped to canonical taxa before structure-organism pair harmonization.

## When NOT to use

- The organism table already contains canonical scientific names directly resolvable without approximate string matching
- Only a small number of known organism names must be matched manually rather than via automated translation
- The paper describes this mechanism but does not independently validate it; do not rely on it without expert review of mapping plausibility

## Inputs

- Cleaned organism table: interim/tables/2_cleaned/organism/cleaned.tsv.gz from the 4_cleaningTaxonomy.R stage
- Common names, vernacular names, TCM names, and organ names to be translated

## Outputs

- SQLite dictionary at interim/dictionaries/organism/otl.sqlite mapping harmonized organism names to Open Tree of Life taxonomy nodes
- Taxonomy node mappings for resolved scientific names

## How to apply

Load the cleaned organism table (interim/tables/2_cleaned/organism/cleaned.tsv.gz) from the lotus-processor 4_cleaningTaxonomy.R stage. Remove organ names of the producing organism to avoid mistranslation, drop common names with three or fewer characters, then translate common names to scientific names via approximate string matching with stringdist. Discard translation pairs where a specific epithet was added (e.g. 'Aloe' → 'Aloe vera') or the name is generic-only; accept a translation only when genus and species names are consistent above 50%. Submit the accepted scientific names to rotl, the R interface to the Open Tree of Life, via the 5_addingOTL.R script, and store the harmonized organism-name-to-taxon mappings as an SQLite dictionary at interim/dictionaries/organism/otl.sqlite.

## Related tools

- **stringdist** (Performs approximate string matching between common/vernacular names and candidate scientific names, with genus/species consistency thresholds at >50%)
- **rotl** (R interface to the Open Tree of Life; resolves accepted scientific names to taxonomy nodes for the mapping table)
- **R** (Language in which the 4_cleaningTaxonomy.R and 5_addingOTL.R harmonization scripts are implemented)

## Examples

```
Rscript 5_addingOTL.R
```

## Evaluation signals

- otl.sqlite is generated and populated at interim/dictionaries/organism/otl.sqlite
- Every cleaned organism name in cleaned.tsv.gz has a mapping entry or is explicitly discarded
- Translation pairs adding a specific epithet (e.g. 'Aloe'→'Aloe vera') or generic-only names are discarded
- Accepted translations show genus and species consistency above 50%
- Mapping plausibility is assessed by expert review

## Limitations

- Correct salts, charged or dimeric molecules in the input data will suffer an unwanted 'sanitization' step — a failure mode of the broader lotus-processor pipeline
- The paper describes this mechanism but does not independently validate it; mapping plausibility is assessed by expert review
- Some correct entries are removed and some incorrect ones kept; the exact count is not given
- The pipeline is heavy, includes many dependencies, and is convoluted

## Evidence

- [readme] Loading the cleaned organism table and approximating string matching: [evidence span withheld — anchor status: external_doc]
- [readme] Roughly aligned: [evidence span withheld — anchor status: external_doc]
- [methods] Organism harmonization in LOTUS pipeline: [evidence span withheld — anchor status: not_found]
- [methods] Common names translated via stringdist with >50% consistency: [evidence span withheld — anchor status: not_found]
- [methods] Output stored in SQLite dictionary: [evidence span withheld — anchor status: not_found]
