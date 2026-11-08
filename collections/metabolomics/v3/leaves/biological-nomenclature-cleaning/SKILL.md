---
name: biological-nomenclature-cleaning
description: Use when an organism table from aggregated NP databases contains mixed
  scientific names, common names, or TCM names that must be mapped to a unified taxonomy
  before building structure-organism pairs.
when_to_use_negative:
- Organism names are already resolved to canonical scientific names or OTT/taxon IDs
  — no string matching is needed
- The task is chemical structure sanitization (standardizing, fragment removal, uncharging
  via MolVS) rather than organism nomenclature cleaning
- Vernacular name translation requires dictionaries or candidate scientific names
  that are not available for the language/domain of the input strings
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: stringdist
  role: Approximate string matching to translate common organism names to candidate
    scientific names, with genus/species consistency assessed at >50%.
- name: rotl
  role: R interface to the Open Tree of Life used to resolve accepted scientific names
    to unified taxonomy nodes.
- name: lotus-processor
  role: Repository hosting the R scripts (4_cleaningTaxonomy.R, 5_addingOTL.R) that
    implement the nomenclature cleaning and OTL mapping stages.
  repo: https://github.com/lotusnprod/lotus-processor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7554/elife.70780
    title: The LOTUS initiative for open knowledge management in natural products
      research
  source_content_hash: sha256:b8824dde04ed186f3f384d70b7b95a201d885cc2e35f9e3385a7f2a275e886e8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/lotusnprod/lotus-processor
  tools:
  - R
  - lotus-processor
  - rotl
  - stringdist
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# biological-nomenclature-cleaning

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Harmonizes heterogeneous organism name strings (scientific, vernacular, TCM) from natural products source databases into standardized Open Tree of Life taxonomy nodes, as implemented in the LOTUS curation pipeline via approximate string matching (stringdist) and taxonomy resolution (rotl).

## When to use

Use when an organism table from aggregated NP databases contains mixed scientific names, common names, or TCM names that must be mapped to a unified taxonomy before building structure-organism pairs.

## When NOT to use

- Organism names are already resolved to canonical scientific names or OTT/taxon IDs — no string matching is needed
- The task is chemical structure sanitization (standardizing, fragment removal, uncharging via MolVS) rather than organism nomenclature cleaning
- Vernacular name translation requires dictionaries or candidate scientific names that are not available for the language/domain of the input strings

## Inputs

- Cleaned organism name table (TSV, gzipped), e.g. interim/tables/2_cleaned/organism/cleaned.tsv.gz
- Aggregated vernacular/TCM-to-scientific name dictionaries
- Original heterogeneous organism name strings from source NP databases (KNApSAcK, NAPRALERT, NPAtlas, etc.)

## Outputs

- SQLite dictionary of harmonized organism-name-to-taxon mappings at interim/dictionaries/organism/otl.sqlite
- Unified taxonomy assignments (Open Tree of Life nodes) for each cleaned organism name

## How to apply

Load the cleaned organism table (interim/tables/2_cleaned/organism/cleaned.tsv.gz) produced by the 4_cleaningTaxonomy.R stage in R. First remove organ names of the producing organism to avoid mistranslation, and drop common names with three or fewer characters. Apply approximate string matching with the stringdist package to translate common names to scientific names, discarding translation pairs where a specific epithet was added (e.g. 'Aloe' -> 'Aloe vera') or the name is generic-only, and accepting a translation only when genus and species names are consistent above 50%. Resolve the accepted scientific names to taxonomy nodes using rotl, the R interface to the Open Tree of Life, via the 5_addingOTL.R script, and store the resulting name-to-taxon mappings as an SQLite dictionary at interim/dictionaries/organism/otl.sqlite.

## Related tools

- **stringdist** (Approximate string matching to translate common organism names to candidate scientific names, with genus/species consistency assessed at >50%.)
- **rotl** (R interface to the Open Tree of Life used to resolve accepted scientific names to unified taxonomy nodes.)
- **lotus-processor** (Repository hosting the R scripts (4_cleaningTaxonomy.R, 5_addingOTL.R) that implement the nomenclature cleaning and OTL mapping stages.) — https://github.com/lotusnprod/lotus-processor

## Examples

```
Rscript 5_addingOTL.R
```

## Evaluation signals

- interim/dictionaries/organism/otl.sqlite is generated and populated with mappings for the cleaned organism names
- Common names with three or fewer characters are absent from the translation input
- No translation pair remains where a specific epithet was added (e.g. 'Aloe' -> 'Aloe vera') or the name is generic-only
- Accepted translations show genus and species name consistency above 50%
- Mapping plausibility is confirmed by expert review of the harmonized name-to-taxon pairs

## Limitations

- The paper describes this mechanism but does not independently validate it; acceptance relies on otl.sqlite being populated and expert plausibility review
- The current processing pipeline still removes a lot of correct entries and misses or induces some incorrect ones
- Vernacular name-to-structure/name translations rely on predefined rules that do not cover all cases and can commonly lead to incorrect translations

## Evidence

- [results] the translation pair was discarded (for example, “Aloe” translated in “Aloe vera” was discarded): "the translation pair was discarded (for example, “Aloe” translated in “Aloe vera” was discarded)"
- [results] If both the species and genus names were consistent at more than 50%, they were considered consistent overall: "If both the species and genus names were consistent at more than 50%, they were considered consistent overall"
- [results] Organ names of the producing organism were removed to avoid wrong translation: "Organ names of the producing organism were removed to avoid wrong translation"
- [results] only common names with more than three characters were kept: "only common names with more than three characters were kept"
- [readme] LOTUS is a comprehensive collection of documented structure-organism pairs.: "LOTUS is a comprehensive collection of documented structure-organism pairs."
