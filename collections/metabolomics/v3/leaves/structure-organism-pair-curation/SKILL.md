---
name: structure-organism-pair-curation
description: Use when a structure database's organism/biological-source field is free
  text and needs detection of scientific names, synonym resolution, and backbone-standardized
  taxonomy before metabolite annotation or scoring.
when_to_use_negative:
- When the organism metadata is already standardized to accepted names against a taxonomy
  backbone — no gnfinder detection or lineage resolution is needed.
- When the analysis only needs spectral similarity scoring and explicitly does not
  use taxonomic information (the paper notes taxonomically informed scoring mostly
  benefits specialized, not ubiquitous, metabolites).
- When the source database cannot be fully downloaded (e.g., KNApSAcK is 'accessible
  online but not downloadable'), making homogeneous taxonomy resolution of the full
  pair table impossible.
edam_operation: http://edamontology.org/operation_3280
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3050
tools:
- name: gnfinder
  role: Detects scientific names and synonyms in the biological-source free text and
    matches them against the Global Names index services
  repo: https://github.com/gnames/gnfinder
- name: Global Names index services
  role: Resolve detected names against the Catalogue of Life taxonomy backbone with
    full lineage to species level
- name: taxo_scorer
  role: 'Downstream consumer: the curated structure-organism table feeds taxonomically
    informed scoring and re-ranking of annotation candidates'
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained repository containing the full curation and annotation workflow
    this curation step belongs to
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:f7c216722509b0e09729ae40d86821756adab370064cc594fa635ba16e049d64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - Global Names index services
  - gnfinder
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_3280
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# structure-organism-pair-curation

## Summary

Curates free-text biological source fields (e.g., from the Dictionary of Natural Products) into accepted scientific names with fully resolved taxonomy, producing a clean one-structure/one-organism-per-row table for taxonomically informed metabolite annotation.

## When to use

Use when a structure database's organism/biological-source field is free text and needs detection of scientific names, synonym resolution, and backbone-standardized taxonomy before metabolite annotation or scoring.

## When NOT to use

- When the organism metadata is already standardized to accepted names against a taxonomy backbone — no gnfinder detection or lineage resolution is needed.
- When the analysis only needs spectral similarity scoring and explicitly does not use taxonomic information (the paper notes taxonomically informed scoring mostly benefits specialized, not ubiquitous, metabolites).
- When the source database cannot be fully downloaded (e.g., KNApSAcK is 'accessible online but not downloadable'), making homogeneous taxonomy resolution of the full pair table impossible.

## Inputs

- Dictionary of Natural Products (DNP) biological-source free-text field (DNP v 27.1 in the paper)
- Catalogue of Life taxonomy backbone (accessed via Global Names index services)

## Outputs

- CSV table with one structure/one-source per row, taxonomy normalized into rank columns (species, genus, family)
- 219,800 entries with accepted scientific names resolved to Catalogue of Life

## How to apply

Load the DNP biological-source free-text field and run gnfinder with English-language detection enabled to detect organism names and their synonyms. Resolve the detected names against the Global Names index services, using the Catalogue of Life as the preferred taxonomy backbone. Retain only entries whose biological source resolves against Catalogue of Life with a complete lineage down to the species level, since downstream taxonomic distance scoring uses species, genus, and family granularity. Normalize the resolved lineage into rank columns and build the one-structure/one-source-per-row table. Save the curated table as CSV and confirm the paper-reported count of 219,800 curated entries.

## Related tools

- **gnfinder** (Detects scientific names and synonyms in the biological-source free text and matches them against the Global Names index services) — https://github.com/gnames/gnfinder
- **Global Names index services** (Resolve detected names against the Catalogue of Life taxonomy backbone with full lineage to species level)
- **taxo_scorer** (Downstream consumer: the curated structure-organism table feeds taxonomically informed scoring and re-ranking of annotation candidates) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained repository containing the full curation and annotation workflow this curation step belongs to) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
gnfinder --verify input/dnp_biological_source.txt --columns 1,2
```

## Evaluation signals

- Output CSV exists and parses without errors.
- Curated row set matches the paper-reported 219,800 entries with accepted scientific names.
- Every retained entry has a taxonomy resolved against Catalogue of Life with a complete lineage down to the species level.
- Taxonomy is normalized into rank columns (species, genus, family) suitable for the downstream taxo_scorer weights (0.81 family, 1.62 genus, 2.55 species for ISDB-DNP).
- Table enforces one-structure/one-source-per-row; duplicate structures/stereoisomers were removed by keeping distinct InChIKey first-layer (first 14 characters).

## Limitations

- Entries whose biological source does not resolve against Catalogue of Life or lacks a complete lineage to species level are discarded.
- The resulting database fills a gap: no publicly available, downloadable databases compiling structures with fully and homogeneously resolved taxonomy existed at the time.
- Taxonomically informed scoring built on this curation mostly benefits specialized metabolites, not ubiquitous molecules.

## Evidence

- [discussion] gnfinder detection step: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] resolution step: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] species-level filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] preferred backbone: "The preferred taxonomy backbone was set to be Catalogue of Life"
- [discussion] no resolved databases existed: "such DBs are not publicly available and downloadable at the moment"
