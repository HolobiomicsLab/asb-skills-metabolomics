---
name: taxonomy-backbone-name-resolution
description: Use when a metabolite or natural-products database stores organism information
  as free text (e.g., DNP Biological Source field) and you need accepted scientific
  names with a resolved, species-complete lineage for downstream taxonomic scoring.
when_to_use_negative:
- The input is already structured taxonomy (e.g., KNApSAcK species-metabolite mappings
  or pre-resolved rank columns) rather than free text — gnfinder name detection is
  unnecessary
- You only need names detected without backbone resolution, e.g., simple name harvesting
  where Catalogue of Life lineage completeness is not required
- Your analysis does not require species-level completeness (e.g., only family-level
  scoring), since this skill explicitly discards entries whose lineage is incomplete
  down to the species level
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: gnfinder
  role: Detects scientific names (and synonyms) in the free-text biological source
    strings and matches them against Global Names index services
  repo: https://github.com/gnames/gnfinder
- name: Global Names index services
  role: Resolution service that maps detected names to the Catalogue of Life taxonomy
    backbone and returns lineage
- name: Catalogue of Life
  role: Preferred taxonomy backbone against which biological sources must resolve
    with complete lineage to species level
- name: tima
  role: Downstream pipeline that consumes the normalized taxonomy for taxo_cleaner
    and taxo_scorer steps
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:49f3633b378e575cb184ec336e871fe243fae2d971440e8bd0d9aad4e1633640
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - Catalogue of Life
  - Global Names index services
  - gnfinder
  - tima
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# taxonomy-backbone-name-resolution

## Summary

Detects scientific organism names embedded in free-text biological-source fields (e.g., Dictionary of Natural Products source strings) using gnfinder, then resolves them against a taxonomy backbone (Catalogue of Life) via the Global Names index to produce normalized, rank-structured taxonomy.

## When to use

Use when a metabolite or natural-products database stores organism information as free text (e.g., DNP Biological Source field) and you need accepted scientific names with a resolved, species-complete lineage for downstream taxonomic scoring.

## When NOT to use

- The input is already structured taxonomy (e.g., KNApSAcK species-metabolite mappings or pre-resolved rank columns) rather than free text — gnfinder name detection is unnecessary
- You only need names detected without backbone resolution, e.g., simple name harvesting where Catalogue of Life lineage completeness is not required
- Your analysis does not require species-level completeness (e.g., only family-level scoring), since this skill explicitly discards entries whose lineage is incomplete down to the species level

## Inputs

- Free-text biological source field from the Dictionary of Natural Products (DNP v 27.1)
- Taxonomy backbone: Catalogue of Life, queried through the Global Names index services
- gnfinder detection configuration (English-language detection enabled)

## Outputs

- Curated table with one structure/one biological source per row and accepted scientific names
- Taxonomy normalized into rank columns (species, genus, family) with complete lineage to species level
- CSV file of curated entries (219,800 entries in the reported DNP curation)

## How to apply

Run gnfinder over the free-text biological source field with English-language detection enabled so it detects organism names and their synonyms using its dictionary and NLP approaches. Match the detected names against the Global Names index services with Catalogue of Life set as the preferred taxonomy backbone. Retain only entries whose biological source resolves against Catalogue of Life AND whose lineage is complete down to the species level — partial or unresolved matches are discarded. Normalize the resolved taxonomy into explicit rank columns (species, genus, family) and build the one-structure/one-source-per-row table. In the TIMA context, this curated taxonomy feeds 1_taxo_cleaner.Rmd and then the taxonomically informed scoring (2_taxo_scorer.Rmd), where the optimal weights were 0.81 (family), 1.62 (genus) and 2.55 (species).

## Related tools

- **gnfinder** (Detects scientific names (and synonyms) in the free-text biological source strings and matches them against Global Names index services) — https://github.com/gnames/gnfinder
- **Global Names index services** (Resolution service that maps detected names to the Catalogue of Life taxonomy backbone and returns lineage)
- **Catalogue of Life** (Preferred taxonomy backbone against which biological sources must resolve with complete lineage to species level)
- **tima** (Downstream pipeline that consumes the normalized taxonomy for taxo_cleaner and taxo_scorer steps) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
gnfinder find dnp_biological_source.txt --lang=eng
```

## Evaluation signals

- Output CSV exists and is parseable, and the curated row count matches the paper-reported 219,800 DNP entries
- Every retained entry has taxonomy resolved against Catalogue of Life with a complete lineage down to species level
- Taxonomy is normalized into distinct rank columns (species, genus, family) supporting one-structure/one-source-per-row granularity
- All retained biological sources carry an accepted scientific name (synonyms resolved away by Global Names index)

## Limitations

- Filtering to species-level-complete Catalogue of Life lineages discards sources that resolve only partially, biasing the curated set toward well-resolved taxa
- Detection depends on gnfinder's dictionary/NLP approaches over free text, so unorthodox source strings or misspellings may be missed or mis-detected
- The resolved names' fitness is dependent on the Currency of the Catalogue of Life backbone version used

## Evidence

- [discussion] gnfinder detection: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] Global Names resolution: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] species-level filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] backbone choice: "The preferred taxonomy backbone was set to be Catalogue of Life"
- [readme] gnfinder mechanism: [evidence span withheld — anchor status: external_doc]
