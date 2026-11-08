---
name: catalogue-of-life-taxonomic-resolution-filtering
description: Use when biological-source metadata contains free-text organism names
  that must be resolved against the Catalogue of Life and filtered to species-level
  completeness before joining structures to sources.
when_to_use_negative:
- The goal is scoring or re-ranking annotation candidates (taxo_scorer), not building
  a taxonomically filtered benchmarking set.
- Entries with genus- or family-level-only taxonomic information are acceptable —
  this filter would wrongly discard them by design.
- Input is a feature table or spectra with no biological-source metadata, so no organism
  names can be resolved.
tools:
- name: gnfinder
  role: Detects scientific names in free-text biological-source fields and matches
    them against the Global Names index services using the Catalogue of Life backbone
  repo: https://github.com/gnames/gnfinder
- name: R / data.table
  role: Joins the taxonomically resolved source table to the GNPS spectral data on
    the short InChIKey and applies species-level filtering
- name: tima
  role: Repository containing the maintained workflow that performs taxonomically
    informed annotation, including taxonomic resolution and cleaning steps
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:02438b90e5c64758dc4a7727860f9fddafd51b899f5cb3ea42b9dc3edee963da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - Catalogue of Life
  - R / data.table
  - gnfinder
  - tima
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Catalogue of Life taxonomic resolution and filtering

## Summary

Resolves organism names extracted from biological-source metadata against the Catalogue of Life backbone and retains only entries whose taxonomy is complete down to the species level, producing the structure–source table used to build the 2,107-entry benchmarking dataset.

## When to use

Use when biological-source metadata contains free-text organism names that must be resolved against the Catalogue of Life and filtered to species-level completeness before joining structures to sources.

## When NOT to use

- The goal is scoring or re-ranking annotation candidates (taxo_scorer), not building a taxonomically filtered benchmarking set.
- Entries with genus- or family-level-only taxonomic information are acceptable — this filter would wrongly discard them by design.
- Input is a feature table or spectra with no biological-source metadata, so no organism names can be resolved.

## Inputs

- Curated DNP-derived structure–biological-source table with free-text taxonomic information
- Filtered GNPS spectral library dataset (40,138 entries, 8,558 unique structures)
- Catalogue of Life taxonomy backbone (via Global Names index services)

## Outputs

- Taxonomically resolved structure–biological-source table filtered to species level
- Benchmarking dataset of 2,107 distinct entries exported as an .mgf spectral file
- Associated sample/source metadata exported as a .tsv file

## How to apply

Extract taxonomic names from the free-text biological-source fields using gnfinder, which detects scientific names and matches them against the Global Names index services with Catalogue of Life as the preferred taxonomy backbone. Retain only entries whose biological source is resolved against the Catalogue of Life AND complete down to the species level — stricter resolutions (genus/family only) are discarded. Harmonize the resulting structure–source table with the GNPS spectral library entries by the short InChIKey (first 14 characters) so structures match across the two sides, then join on the short InChIKey, keeping one unique biological source per structure and deduplicating stereoisomers per the paper's convention of distinct first-14-character InChIKeys. The merge of the filtered DNP-derived table with the filtered GNPS spectral dataset (40,138 entries covering 8,558 unique structures) must yield exactly 2,107 distinct entries, written as an .mgf spectral file plus a .tsv of sample/source metadata.

## Related tools

- **gnfinder** (Detects scientific names in free-text biological-source fields and matches them against the Global Names index services using the Catalogue of Life backbone) — https://github.com/gnames/gnfinder
- **R / data.table** (Joins the taxonomically resolved source table to the GNPS spectral data on the short InChIKey and applies species-level filtering)
- **tima** (Repository containing the maintained workflow that performs taxonomically informed annotation, including taxonomic resolution and cleaning steps) — https://github.com/taxonomicallyinformedannotation/tima

## Evaluation signals

- The final merged table contains exactly 2,107 distinct entries, matching the reported size of the benchmarking dataset.
- Every retained entry has biological source information resolved against the Catalogue of Life and complete down to the species level.
- Both output files (.mgf spectra and .tsv metadata) exist and are well-formed, with entry counts matching between them.
- Duplicate structures and stereoisomers are removed so that all retained InChIKeys are distinct in their first 14 characters.
- The pre-merge spectral side can be cross-checked at 40,138 entries covering 8,558 unique structures.

## Limitations

- Requiring complete species-level resolution against the Catalogue of Life discards entries with only genus- or family-level taxonomic information.
- Only a single, unique biological source per structure is kept, collapsing multi-source natural products.
- Alternative species–metabolite resources are limited: KNApSAcK is accessible online but not downloadable, and no publicly downloadable databases with fully and homogeneously resolved taxonomy existed at the time.
- Deduplication collapses stereoisomers by convention (first 14 characters of the InChIKey), which loses stereochemical distinction.

## Evidence

- [discussion] Species-level resolution filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] Catalogue of Life as backbone: "The preferred taxonomy backbone was set to be Catalogue of Life"
- [discussion] gnfinder name extraction: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] gnfinder name matching: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] Benchmarking dataset size: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [readme] gnfinder capability: [evidence span withheld — anchor status: external_doc]
