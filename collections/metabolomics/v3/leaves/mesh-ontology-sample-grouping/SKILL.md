---
name: mesh-ontology-sample-grouping
description: Use when sample metadata from many GC/MS metabolomics studies is free-text,
  inconsistent, or riddled with domain-specific acronyms, and the goal is to run ontologically
  grouped differential analysis (OGDA) or phylogenetic-organization queries across
  studies.
when_to_use_negative:
- When the analysis only needs per-study sample-level grouping and no cross-study
  ontological aggregation is intended.
- When the raw metadata is already standardized to a fixed ontology (e.g., ReDU or
  RefMet) with no free-text variability.
- When the input dataset has fewer than 10 samples per metadata triad, since grouping
  cannot produce robust comparisons.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3174
tools:
- name: BinDiscover
  role: Webtool that consumes the ontology-mapped metadata triads and enables OGDA
    queries across aggregated sample groups
- name: PostgreSQL
  role: Stores the ontology-normalized metadata and averages/medians/ratios in the
    BinDiscover output database
- name: Python
  role: Custom scripts (with SciPy and NetworkX) used to map and aggregate metadata
    onto ontologies and build the adjacency graph
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:a7b3cae6cf4d0ac86d14a6a464bb9c28d349996774ffcc882b3cb52f4cc0d374
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinBase
  - BinDiscover
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# MeSH-ontology sample grouping

## Summary

A metadata-curation step in the BinDiscover pipeline that maps free-text biological sample metadata (organ, species, phenotype, compound class) onto established MeSH and NCBI ontologies so that related metadata combinations become adjacent and can be aggregated for broad comparisons such as 'digestive tract' or 'rodents'.

## When to use

Use when sample metadata from many GC/MS metabolomics studies is free-text, inconsistent, or riddled with domain-specific acronyms, and the goal is to run ontologically grouped differential analysis (OGDA) or phylogenetic-organization queries across studies.

## When NOT to use

- When the analysis only needs per-study sample-level grouping and no cross-study ontological aggregation is intended.
- When the raw metadata is already standardized to a fixed ontology (e.g., ReDU or RefMet) with no free-text variability.
- When the input dataset has fewer than 10 samples per metadata triad, since grouping cannot produce robust comparisons.

## Inputs

- Free-text biological metadata triads (species, biospecimen, phenotype) from BinBase study-design database
- MeSH ontology and NCBI taxonomy term sets
- Misspelled or synonym-variant metadata combinations

## Outputs

- Ontologically normalized metadata triads with MeSH/NCBI term IDs
- Ontology adjacency (belongs-to) graph linking related metadata combinations
- Set of aggregated metadata combinations per generic query term

## How to apply

Collect the raw metadata triads (species, biospecimen, phenotype) exported from the BinBase study-design database, then curate and combine them to map each term to established ontologies — MeSH for organs/biofluids and compounds, NCBI taxonomy for species — correcting misspellings and merging synonymic combinations so that downstream grouping does not split semantically identical classes. Use ontology proximity ('belongs to' edges) to create adjacency among metadata combinations, enabling generic terms like 'digestive tract' or 'amino acids' to expand into the full set of related metadata combinations for a query. Grouping must preserve the triad identity so that per-combination sample counts can later be filtered (triads with >20% failed FAME samples or fewer than 10 samples are dropped).

## Related tools

- **BinDiscover** (Webtool that consumes the ontology-mapped metadata triads and enables OGDA queries across aggregated sample groups)
- **PostgreSQL** (Stores the ontology-normalized metadata and averages/medians/ratios in the BinDiscover output database)
- **Python** (Custom scripts (with SciPy and NetworkX) used to map and aggregate metadata onto ontologies and build the adjacency graph)

## Examples

```
python binmetadata_normalize.py --input binbase_metadata_triads.tsv --ontology mesh --output normalized_triads.json
```

## Evaluation signals

- The curated metadata set contains no unmapped free-text terms; every triad is linked to a MeSH or NCBI term.
- Generic query terms (e.g., 'digestive tract') expand to the full set of related metadata combinations without excluding semantically identical but differently spelled variants.
- Sample-count filtering downstream retains triads with ≥10 samples and ≤20% failed FAME samples.
- OGDA output matches known category groupings (e.g., 'rodents', 'amino acids') without splitting them into separate classes.

## Limitations

- Domain-specific acronyms in biologist-delivered sample lists were intractable to compile retrospectively across ~2000 studies.
- Inconsistent metadata terminology in the miniX study design DB and omission of fine-grained biological study design details caused grouping errors.
- A graph-based metadata capture interface was abandoned because biologists found it frustrating and overwhelming.

## Evidence

- [methods] Metadata mapping to ontologies: "we curated and combined metadata combinations to map metadata to established ontologies and to correct for misspellings"
- [abstract] Ontology-based aggregation: "we emphasize the use of ontologies to aggregate large groups of samples based on the proximity of their metadata within these ontologies"
- [intro] Metadata loss when translating: "Too much biological metadata is lost in translating either text sources (as in HMDB) or cryptic and unstructured sample/treatment naming schemes"
- [discussion] Acronyms: "domain-specific acronyms were used that were intractable to compile retrospectively throughout the diversity of 2000 different studies in our GC/MS database"
- [intro] OGDA transform: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs"
