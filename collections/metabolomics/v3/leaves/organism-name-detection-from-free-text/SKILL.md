---
name: organism-name-detection-from-free-text
description: Use when natural product or metabolomics records contain unstructured
  free-text biological source fields that must be converted into scientific organism
  names before taxonomic resolution.
when_to_use_negative:
- The biological source field already contains resolved, accepted scientific names
  with complete taxonomy — no name detection is needed.
- The target taxonomy backbone is not Catalogue of Life, since this workflow retains
  only entries resolved against Catalogue of Life down to species level.
- Text is short controlled vocabulary or organism codes rather than free prose where
  name detection adds no value.
edam_operation: http://edamontology.org/operation_3280
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3050
tools:
- name: gnfinder
  role: Detects scientific names in free-text biological source content and matches
    them against the Global Names index
  repo: https://github.com/gnames/gnfinder
- name: taxo_scorer / tima pipeline
  role: Consumes the detected and resolved organism names for taxonomically informed
    scoring of metabolite annotation candidates
  repo: https://github.com/oolonek/taxo_scorer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:4bc2a6a664a7d6570be8a89bd588be173963e8982e1984960c9b5e22878a704f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - Global Names index services
  - gnfinder
  - taxo_scorer / tima pipeline
  edam_operation: http://edamontology.org/operation_3280
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Organism name detection from free text

## Summary

Detects scientific organism names in free-text biological-source strings (e.g. the DNP source field) using gnfinder, then matches detected names against the Global Names index to attach synonyms and prepare them for taxonomic resolution.

## When to use

Use when natural product or metabolomics records contain unstructured free-text biological source fields that must be converted into scientific organism names before taxonomic resolution.

## When NOT to use

- The biological source field already contains resolved, accepted scientific names with complete taxonomy — no name detection is needed.
- The target taxonomy backbone is not Catalogue of Life, since this workflow retains only entries resolved against Catalogue of Life down to species level.
- Text is short controlled vocabulary or organism codes rather than free prose where name detection adds no value.

## Inputs

- Free-text biological source field from the Dictionary of Natural Products (DNP)
- gnfinder program with English-language detection enabled
- Global Names index services for name matching

## Outputs

- Detected scientific organism names with synonyms per source string
- Taxonomically resolved biological source table with accepted names (219,800 curated entries)
- Curated CSV of one-structure/one-source-per-row DNP entries

## How to apply

Run gnfinder (https://github.com/gnames/gnfinder) over the free-text biological source field to detect scientific names and their synonyms, relying on its dictionary and NLP-based detection. Enable English-language detection so vernacular phrases are handled when scanning the DNP source strings. Then match the detected names against the Global Names index services, which returns name candidates and synonyms that feed the downstream resolution against Catalogue of Life. Only entries whose sources resolve against Catalogue of Life and are complete down to the species level should be kept, as the paper does. Output is a table of curated biological sources with accepted scientific names (219,800 entries in the paper's curation), usable as the taxonomic backbone for taxonomically informed annotation scoring.

## Related tools

- **gnfinder** (Detects scientific names in free-text biological source content and matches them against the Global Names index) — https://github.com/gnames/gnfinder
- **taxo_scorer / tima pipeline** (Consumes the detected and resolved organism names for taxonomically informed scoring of metabolite annotation candidates) — https://github.com/oolonek/taxo_scorer

## Examples

```
gnfinder --language en --verify-names all sources.txt > detected_names.tsv
```

## Evaluation signals

- Output CSV is parseable and contains one row per retained structure–source pair.
- Count of curated entries matches the paper-reported 219,800 DNP entries with accepted scientific names.
- Every retained entry resolves against Catalogue of Life with a complete lineage down to species level.
- Detected names are matched against Global Names index before resolution, so synonym variants converge to accepted names.
- Biological source strings that fail resolution (no Catalogue of Life match or incomplete lineage) are excluded from the final table.

## Limitations

- Only entries whose biological source information resolves against the Catalogue of Life and is complete down to the species level are retained; sources that fail resolution are dropped.
- Vernacular or ambiguous free-text phrases in the source field may not be detected by gnfinder's dictionary/NLP approach.
- The retained count is dataset-dependent: 219,800 reflects DNP v27.1 curation and will differ for other source databases.

## Evidence

- [discussion] gnfinder used for detection of organism names and synonyms: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] gnfinder matches detected names against Global Names index services: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] Filter to Catalogue of Life-resolved sources complete to species level: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [readme] gnfinder is a fast finder of scientific names using dictionary and NLP approaches: [evidence span withheld — anchor status: external_doc]
