---
name: database-taxonomy-structuring
description: Use when building or documenting a multiomics database and you need to
  enumerate its constituent data types (e.g. epigenomics marks), organizational modules,
  and user-facing functionalities into a structured schema.
when_to_use_negative:
- Input is raw omics data files (FASTQ, VCF) rather than textual descriptions of database
  organization — data curation, not taxonomy structuring, is needed.
- The database already publishes a machine-readable schema or API spec; reuse it instead
  of re-deriving the taxonomy from prose.
- Only a single data type is in scope (e.g. one histone mark), where a full module/functionality
  taxonomy adds no value.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Parse article/README text, extract and enumerate taxonomy terms, and serialize
    them into a JSON file with keys for epigenomics, modules, and functionalities
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:3dd71a4f12e475aa9f66cd36a39aeedea3b63910f52e88524c9290e6ed508f1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3071
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# database-taxonomy-structuring

## Summary

Structures a multiomics database's content into an explicit taxonomy of data types, modules, and functionalities, as done for GutUDB's eight omics types, four modules (SPECIES, DISEASES, OMICS, THERAPY), and four core functionalities (Browse, Query, Visualization, Download).

## When to use

Use when building or documenting a multiomics database and you need to enumerate its constituent data types (e.g. epigenomics marks), organizational modules, and user-facing functionalities into a structured schema.

## When NOT to use

- Input is raw omics data files (FASTQ, VCF) rather than textual descriptions of database organization — data curation, not taxonomy structuring, is needed.
- The database already publishes a machine-readable schema or API spec; reuse it instead of re-deriving the taxonomy from prose.
- Only a single data type is in scope (e.g. one histone mark), where a full module/functionality taxonomy adds no value.

## Inputs

- Article full text or abstract describing database organization (GutUDB, DOI 10.1002/imt2.195)
- Repository README (github.com/Ansanqi/GutUDB) with database structure statements
- Reported counts per category (e.g. 9 million profiles, 58,970 genes, 72,248 SNPs)

## Outputs

- JSON taxonomy file with keys for epigenomics types, histone marks, modules, and functionalities
- Cross-checked category-to-count mapping table

## How to apply

Parse the article text and repository README to locate every statement describing data types and database organization. Extract the enumerated categories — for GutUDB, the three epigenomics types (DNA methylation, histone modification across six marks: H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3, and chromosome structure), the eight omics data set types behind ~9 million profiles, the four modules (SPECIES, DISEASES, OMICS, THERAPY), and the four functionalities (Browse, Query, Visualization, Download). Cross-check each extracted term against the abstract's reported counts (e.g. 260,790 disease-gene associations, 21,984 drug-disease interactions with 6281 chemical compounds, 393 traditional medicinal herbs, 22 probiotics) to confirm no term is missing and every count has a category. Write the enumerated terms into a structured file (e.g. JSON with keys for epigenomics, modules, functionalities) so downstream agents can query the taxonomy programmatically.

## Related tools

- **Python** (Parse article/README text, extract and enumerate taxonomy terms, and serialize them into a JSON file with keys for epigenomics, modules, and functionalities) — https://github.com/Ansanqi/GutUDB

## Examples

```
json.dump({'epigenomics':['DNA methylation','H3K27me3','H3K27ac','H3K36me3','H3K4me1','H3K4me3','H3K9me3','chromosome structure'],'modules':['SPECIES','DISEASES','OMICS','THERAPY'],'functionalities':['Browse','Query','Visualization','Download']}, open('gutudb_taxonomy.json','w'))
```

## Evaluation signals

- The six histone marks (H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3) all appear in the output epigenomics key.
- All four modules (SPECIES, DISEASES, OMICS, THERAPY) and all four functionalities (Browse, Query, Visualization, Download) are present in the output.
- Every extracted category has a corresponding count cross-checked against the abstract (e.g. 260,790 disease-gene associations, 21,984 drug-disease interactions).
- Output JSON validates against the declared schema (keys: epigenomics, modules, functionalities) and parses without error.
- No category mentioned in the abstract or README is absent from the taxonomy (zero missing-term check).

## Limitations

- Counts are taken from the article's abstract and are point-in-time figures; the repository README shows no changelog, so updates to the database may silently invalidate the extracted numbers.
- Pre-resolved metadata flags doi, title, authors, year, and journal as missing from the abstract section itself, so metadata must come from external sources rather than the parsed text.
- Taxonomy terms are extracted from prose; ambiguity in the source text (e.g. overlapping module/omics categories) can lead to mis-binned terms.

## Evidence

- [other] six histone-modification marks and eight omics taxonomy: [evidence span withheld — anchor status: not_found]
- [abstract] four core functionalities integrated: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [abstract] scale of omics coverage: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] THERAPY module counts: "GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics"
