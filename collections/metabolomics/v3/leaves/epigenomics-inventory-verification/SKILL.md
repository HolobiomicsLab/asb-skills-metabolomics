---
name: epigenomics-inventory-verification
description: Use when a database paper claims a fixed inventory of omics types, histone
  marks, or internal modules (e.g. 'eight types of omics data sets') and the extracted
  term list must be reconciled with those reported counts.
when_to_use_negative:
- The goal is primary epigenomic analysis (e.g. peak calling on new ChIP-seq or methylation
  data) rather than verifying a database's self-reported inventory of data types
- The source article reports no abstract-level counts or enumerated taxonomy to cross-check,
  so the verification step has no invariant to test against
- The input is already a structured feature table or completed ontology dump rather
  than unstructured article/README text needing extraction
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Python
  role: Parses article text and README, extracts and cross-checks enumerated terms,
    and writes the verified taxonomy JSON
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:c0d2a0bc83b6b1584680ba0c333c5af47c30166c33a91db8e2bc7176ac6fc12e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# epigenomics-inventory-verification

## Summary

Extracts and verifies a database article's enumerated data-type inventory — e.g. GutUDB's six histone-modification marks (H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3), DNA methylation and chromosome structure, plus its module (SPECIES, DISEASES, OMICS, THERAPY) and functionality (Browse, Query, Visualization, Download) taxonomies — by cross-checking extracted terms against the abstract's reported counts.

## When to use

Use when a database paper claims a fixed inventory of omics types, histone marks, or internal modules (e.g. 'eight types of omics data sets') and the extracted term list must be reconciled with those reported counts.

## When NOT to use

- The goal is primary epigenomic analysis (e.g. peak calling on new ChIP-seq or methylation data) rather than verifying a database's self-reported inventory of data types
- The source article reports no abstract-level counts or enumerated taxonomy to cross-check, so the verification step has no invariant to test against
- The input is already a structured feature table or completed ontology dump rather than unstructured article/README text needing extraction

## Inputs

- Database article full text or abstract (e.g. GutUDB, DOI 10.1002/imt2.195)
- Repository README from the project's GitHub repo (e.g. Ansanqi/GutUDB)
- Pre-resolved metadata (DOI, title, authors, year, journal) for provenance fields

## Outputs

- JSON file with keys for epigenomics (histone marks, DNA methylation, chromosome structure), modules, and functionalities
- Verified count reconciliation (extracted term counts vs. abstract-reported counts)
- List of missing or unverifiable metadata signals (e.g. absent title/authors/year/journal fields, absent changelog)

## How to apply

Parse the article text and the repository README (e.g. github.com/Ansanqi/GutUDB) to locate every statement describing epigenomic content and database organization. Enumerate each class explicitly — for GutUDB this means the six histone marks H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3, plus DNA methylation and chromosome structure under epigenomics, the SPECIES/DISEASES/OMICS/THERAPY modules, and the Browse/Query/Visualization/Download functionalities. Cross-check the enumerated terms against the abstract's headline counts — 'eight types of omics data sets' covering 56 intestinal diseases across six species — to confirm no term is missing or spuriously added; counts that only appear as approximations (e.g. 'approximately 9 million generated profiles') are treated as sanity bounds, not exact invariants. Write the verified terms into a JSON file keyed by epigenomics, modules, and functionalities so downstream agents can consume the inventory directly.

## Related tools

- **Python** (Parses article text and README, extracts and cross-checks enumerated terms, and writes the verified taxonomy JSON)

## Examples

```
python extract_taxonomy.py --source README.md --article gutudb_abstract.txt --keys epigenomics,modules,functionalities --out taxonomy.json
```

## Evaluation signals

- The epigenomics key contains exactly the six marks H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3 plus DNA methylation and chromosome structure — no extra or missing marks
- The modules key equals SPECIES, DISEASES, OMICS, THERAPY and the functionalities key equals Browse, Query, Visualization, Download
- Extracted term counts are reconciled against abstract claims such as 'eight types of omics data sets' with 56 intestinal diseases across six species
- JSON schema check: exactly the keys epigenomics, modules, functionalities are present and non-empty
- Missing metadata signals (title, authors, year, journal, changelog) are reported rather than silently dropped

## Limitations

- Some headline counts are approximations only ('approximately 9 million generated profiles'), so reconciliation against them yields bounds, not exact matches
- Article-derived metadata may be incomplete — the source card flags doi, title, authors, year, and journal as absent from the abstract text, and the repository README contains no changelog, limiting provenance verification
- Counts extracted from the abstract alone (e.g. 72,248 SNPs, 260,790 disease-gene associations) cannot be independently re-derived without database access, so verification is textual consistency, not empirical validation

## Evidence

- [abstract] four core functionalities integrated into GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [abstract] eight omics types across diseases and species: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
