---
name: synonym-resolution-against-catalogue-of-life
description: Use when a natural products or metabolomics database stores organism
  information as heterogeneous free text that must be converted into accepted scientific
  names with a complete, resolvable lineage for taxonomic scoring.
when_to_use_negative:
- The organism names are already controlled vocabulary or already resolved to accepted
  names with lineage — no curation is needed
- Taxa of interest cannot be resolved against Catalogue of Life (e.g. only entries
  with complete species-level lineage are retained, so unresolved sources will be
  dropped entirely)
- The goal is specimen-level taxonomic assignment from raw sequence data — this skill
  curates database free-text fields, not reads
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: gnfinder
  role: Detects scientific names and synonyms in the free-text biological source field,
    with English-language detection enabled, and matches detected names against Global
    Names index services
  repo: https://github.com/gnames/gnfinder
- name: Global Names index services
  role: 'Resolution backbone: matched names are resolved against the Catalogue of
    Life to obtain accepted names and complete lineage'
- name: taxo_scorer
  role: 'Downstream consumer: consumes the resolved taxonomy for taxonomically informed
    scoring of metabolite annotations'
  repo: https://github.com/oolonek/taxo_scorer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:9121f4de2e1e752c92a0b5aef0499046b6e774fd9c79c1d1b4cf7d5fa57c27bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gnames/gnfinder
  tools:
  - Global Names index services
  - gnfinder
  - taxo_scorer
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# synonym-resolution-against-catalogue-of-life

## Summary

Curates free-text biological source fields (e.g. the Dictionary of Natural Products organism field) by detecting scientific names and their synonyms with gnfinder and resolving them against the Catalogue of Life backbone via the Global Names index services, yielding normalized taxonomy down to species level.

## When to use

Use when a natural products or metabolomics database stores organism information as heterogeneous free text that must be converted into accepted scientific names with a complete, resolvable lineage for taxonomic scoring.

## When NOT to use

- The organism names are already controlled vocabulary or already resolved to accepted names with lineage — no curation is needed
- Taxa of interest cannot be resolved against Catalogue of Life (e.g. only entries with complete species-level lineage are retained, so unresolved sources will be dropped entirely)
- The goal is specimen-level taxonomic assignment from raw sequence data — this skill curates database free-text fields, not reads

## Inputs

- Free-text biological source field from the Dictionary of Natural Products (DNP v 27.1)
- Catalogue of Life taxonomy backbone (accessed through Global Names index services)

## Outputs

- Curated CSV table with one-structure/one-source-per-row entries carrying accepted scientific names
- Normalized taxonomic rank columns (species, genus, family) suitable for taxonomically informed scoring
- Expected ~219,800 curated entries for the full DNP biological source field

## How to apply

Run gnfinder on the biological source free-text field with English-language detection enabled so it detects organism names and their synonyms using its dictionary and NLP approaches. Match the detected names against the Global Names index services to resolve them to the preferred taxonomy backbone, set here to Catalogue of Life. Retain only entries whose biological source is resolved against Catalogue of Life and whose lineage is complete down to the species level, then normalize the resolved taxonomy into separate rank columns (species, genus, family). Build the one-structure/one-source-per-row table and export it as CSV; on the DNP v 27.1 this curation produced 219,800 entries with accepted scientific names, which serves as the expected count benchmark.

## Related tools

- **gnfinder** (Detects scientific names and synonyms in the free-text biological source field, with English-language detection enabled, and matches detected names against Global Names index services) — https://github.com/gnames/gnfinder
- **Global Names index services** (Resolution backbone: matched names are resolved against the Catalogue of Life to obtain accepted names and complete lineage)
- **taxo_scorer** (Downstream consumer: consumes the resolved taxonomy for taxonomically informed scoring of metabolite annotations) — https://github.com/oolonek/taxo_scorer

## Examples

```
gnfinder find --lang English dnp_biological_source.txt  # then resolve detected names against Catalogue of Life via Global Names index services
```

## Evaluation signals

- Output CSV exists and is parseable without errors
- Row count of the curated table matches the paper-reported 219,800 entries for the DNP biological source field
- Every retained entry resolves against Catalogue of Life with a complete lineage down to the species level
- Taxonomy is normalized into distinct rank columns and the table follows the one-structure/one-source-per-row schema
- Detected synonyms are mapped to a single accepted scientific name rather than kept as separate rows

## Limitations

- Entries whose biological source does not resolve against Catalogue of Life or lacks a complete species-level lineage are discarded, so unresolved or ambiguous source strings are lost
- The backbone choice (Catalogue of Life) determines accepted names; taxa absent from or disagreeing with this backbone will fail resolution
- The curated table is only as current as the Global Names index and Catalogue of Life versions used at resolution time

## Evidence

- [discussion] species-level lineage filter: "only entries having biological source information resolved against the Catalogue of Life and complete down to the species level were retained"
- [discussion] gnfinder name detection and resolution: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [discussion] Global Names index matching: "gnfinder allows to match the detected names against the Global Names index services"
- [readme] gnfinder method description: [evidence span withheld — anchor status: external_doc]
