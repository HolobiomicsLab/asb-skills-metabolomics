---
name: species-level-taxonomy-lineage-completion
description: Use when a natural products database stores organism/origin information
  as heterogeneous free text and you need one accepted scientific name per row with
  complete taxonomy down to the species level.
when_to_use_negative:
- The organism field already contains validated scientific names or structured rank
  columns — no name detection/resolution is needed.
- The goal is to re-rank candidate metabolite annotations by taxonomic distance (that
  is the separate taxo_scorer/tima scoring step, which consumes this skill's output).
- Entries whose biological source cannot resolve against Catalogue of Life or is not
  complete to the species level are acceptable — this skill deliberately discards
  them.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: gnfinder
  role: Detects scientific names (and synonyms) in the free-text biological source
    field using dictionary and NLP approaches, with English-language detection enabled
  repo: https://github.com/gnames/gnfinder
- name: Global Names index services
  role: Resolve detected names against the Catalogue of Life taxonomy backbone to
    obtain accepted names and full lineage
- name: tima
  role: Repository containing the taxo_cleaner step (1_taxo_cleaner.Rmd) that cleans
    and standardizes taxonomic outputs into rank columns
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:303ecea21f943cb8341b8b4abc5b2087d928189731e0e892089f8043a941657c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - Global Names index services
  - gnfinder
  - tima
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# species-level-taxonomy-lineage-completion

## Summary

Curates free-text biological source fields (e.g. the Dictionary of Natural Products biological source field) into accepted scientific names with a fully resolved lineage by detecting organism names with gnfinder and matching them against the Catalogue of Life backbone via the Global Names index services.

## When to use

Use when a natural products database stores organism/origin information as heterogeneous free text and you need one accepted scientific name per row with complete taxonomy down to the species level.

## When NOT to use

- The organism field already contains validated scientific names or structured rank columns — no name detection/resolution is needed.
- The goal is to re-rank candidate metabolite annotations by taxonomic distance (that is the separate taxo_scorer/tima scoring step, which consumes this skill's output).
- Entries whose biological source cannot resolve against Catalogue of Life or is not complete to the species level are acceptable — this skill deliberately discards them.

## Inputs

- Dictionary of Natural Products (DNP) free-text biological source field
- gnfinder detection output (detected organism names and synonyms)
- Catalogue of Life taxonomy backbone (via Global Names index services)

## Outputs

- Curated CSV table with accepted scientific names and normalized taxonomy rank columns, complete to species level (219,800 entries for DNP v27.1)
- One-structure/one-source-per-row table usable as taxonomic metadata for downstream taxonomically informed annotation

## How to apply

Load the biological-source free-text field (in the paper, DNP v27.1 where taxonomic information appears in two fields) and run gnfinder with English-language detection enabled to detect scientific names and their synonyms. Match the detected names against the Global Names index services, setting Catalogue of Life as the preferred taxonomy backbone. Retain only entries whose source resolves against Catalogue of Life with a complete lineage down to the species level; entries failing resolution or missing lower ranks are dropped. Normalize the resolved lineage into explicit rank columns (kingdom/family/genus/species) and build the one-structure/one-source-per-row table, saved as CSV. In the DNP curation this yielded 219,800 curated entries, which is the expected validation count.

## Related tools

- **gnfinder** (Detects scientific names (and synonyms) in the free-text biological source field using dictionary and NLP approaches, with English-language detection enabled) — https://github.com/gnames/gnfinder
- **Global Names index services** (Resolve detected names against the Catalogue of Life taxonomy backbone to obtain accepted names and full lineage)
- **tima** (Repository containing the taxo_cleaner step (1_taxo_cleaner.Rmd) that cleans and standardizes taxonomic outputs into rank columns) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
Rscript -e "rmarkdown::render('1_taxo_cleaner.Rmd')"  # detects names with gnfinder (English detection), resolves vs Catalogue of Life, writes the 219,800-entry CSV
```

## Evaluation signals

- Output CSV exists and is parseable.
- Curated row count equals the paper-reported 219,800 entries for the DNP source data.
- Every retained entry has a biological source resolved against Catalogue of Life with a complete lineage down to the species level.
- Taxonomy is normalized into explicit rank columns (species, genus, family) consistent with the one-structure/one-source-per-row schema.
- Resolved names are accepted scientific names (synonyms replaced by their accepted Catalogue of Life counterparts).

## Limitations

- Only entries resolving against Catalogue of Life and complete to species level are retained, so sources that fail resolution or lack lower-rank lineage are silently excluded.
- Optimized/completed taxonomy is dependent on the chosen backbone; the paper set Catalogue of Life as preferred, and results would differ with another backbone.
- DNP taxonomic information appears in two fields, so free-text curation may still miss context encoded outside the biological source field.

## Evidence

- [discussion] Extraction of taxonomic information from free text with gnfinder: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] Resolution against Global Names index: "gnfinder allows to match the detected names against the Global Names index services"
- [discussion] Species-level lineage filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] gnfinder repository reference: "the gnfinder program (https://github.com/gnames/gnfinder) was used"
- [discussion] Cleaning/standardization script: "1) cleaning and standardization of the outputs (1_taxo_cleaner.Rmd)"
