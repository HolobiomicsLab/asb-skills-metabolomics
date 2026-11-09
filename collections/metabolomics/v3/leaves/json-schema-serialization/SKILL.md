---
name: json-schema-serialization
description: Use when a task card asks you to enumerate a paper's taxonomic terms
  (e.g. histone marks, database modules, functionalities) and emit them as a structured
  JSON file verified against reported counts.
when_to_use_negative:
- Input is already a structured data table (e.g. a gene-expression matrix or feature
  table) rather than prose describing a taxonomy
- The target terms are quantitative results (e.g. 72,248 SNPs or 260,790 disease-gene
  associations) that require numeric extraction and unit handling rather than enumerated
  category serialization
- The article's abstract does not report checkable counts, so the cross-verification
  step of this skill cannot be performed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Parses article/README text, extracts the enumerated terms, cross-checks against
    abstract counts, and writes the JSON file via the json module
- name: GutUDB
  role: Source multiomics database whose epigenomics marks, modules, and functionalities
    constitute the serialized term set
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:26cac4c72d46d93ecb68833436d31d0e1844a146274050865809270a28c8ba57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# json-schema-serialization

## Summary

Extracts enumerated domain terms from a scientific article's text (e.g. GutUDB's six histone marks, four modules, four functionalities), cross-checks them against counts reported in the abstract, and serializes the verified set into a JSON file with explicit keys.

## When to use

Use when a task card asks you to enumerate a paper's taxonomic terms (e.g. histone marks, database modules, functionalities) and emit them as a structured JSON file verified against reported counts.

## When NOT to use

- Input is already a structured data table (e.g. a gene-expression matrix or feature table) rather than prose describing a taxonomy
- The target terms are quantitative results (e.g. 72,248 SNPs or 260,790 disease-gene associations) that require numeric extraction and unit handling rather than enumerated category serialization
- The article's abstract does not report checkable counts, so the cross-verification step of this skill cannot be performed

## Inputs

- Scientific article full text or abstract (e.g. GutUDB, DOI 10.1002/imt2.195)
- Repository README (Ansanqi/GutUDB) describing database organization
- Task card specifying which terms/counts must be extracted

## Outputs

- JSON file with keys for epigenomics (DNA methylation, six histone marks, chromosome structure), modules (SPECIES, DISEASES, OMICS, THERAPY), and functionalities (Browse, Query, Visualization, Download)

## How to apply

Parse the article text and repository README to locate every statement describing the target taxonomy — for GutUDB this means the epigenomics types (DNA methylation, histone modification across six marks, chromosome structure), the four modules (SPECIES, DISEASES, OMICS, THERAPY), and the four functionalities (Browse, Query, Visualization, Download). Extract each term verbatim, including all six histone marks (H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3). Cross-check the extracted sets against the abstract's reported counts — e.g. eight types of omics data sets and 56 intestinal diseases across six species — to confirm no term is missing or hallucinated. Finally write the verified terms into a JSON file keyed by 'epigenomics', 'modules', and 'functionalities' using Python's json module.

## Related tools

- **Python** (Parses article/README text, extracts the enumerated terms, cross-checks against abstract counts, and writes the JSON file via the json module)
- **GutUDB** (Source multiomics database whose epigenomics marks, modules, and functionalities constitute the serialized term set) — https://github.com/Ansanqi/GutUDB

## Examples

```
import json; json.dump({'epigenomics': ['DNA methylation', 'H3K27me3', 'H3K27ac', 'H3K36me3', 'H3K4me1', 'H3K4me3', 'H3K9me3', 'chromosome structure'], 'modules': ['SPECIES', 'DISEASES', 'OMICS', 'THERAPY'], 'functionalities': ['Browse', 'Query', 'Visualization', 'Download']}, open('gutudb_taxonomy.json', 'w'), indent=2)
```

## Evaluation signals

- All six histone marks (H3K27me3, H3K27ac, H3K36me3, H3K4me1, H3K4me3, H3K9me3) appear in the 'epigenomics' key
- All four modules (SPECIES, DISEASES, OMICS, THERAPY) and four functionalities (Browse, Query, Visualization, Download) appear in their respective keys
- Extracted term counts reconcile with abstract-reported figures such as 'eight types of omics data sets' and '56 intestinal diseases across six species'
- Output is valid, loadable JSON (json.load succeeds) with exactly the keys epigenomics, modules, functionalities
- No term appears in the output that cannot be traced to a verbatim statement in the article or README

## Limitations

- Verification depends on the abstract's self-reported counts; if counts change in later GutUDB releases, the cross-check must be re-run
- The article text processed here is abstract-level; terms present only in methods/supplementary sections may be missed unless full text is available
- No changelog was found in the repository, so term drift between database versions cannot be tracked automatically

## Evidence

- [abstract] To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [abstract] GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics: "GutUDB curated 21,984 drug‐disease interactions, including 6281 chemical compounds, 393 traditional medicinal herbs, and 22 probiotics"
