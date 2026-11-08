---
name: tabular-result-assembly
description: Use when a BinDiscover OGDA query yields a list of significant metabolites
  (e.g. 15 hits from a human-digestive-tract-vs-bacteria comparison) that must be
  exported with per-compound annotation columns for downstream enrichment.
when_to_use_negative:
- Input is already a fully annotated feature table with non-empty ontology columns
  — assembling it again adds no information.
- The OGDA query returned a graph-based or single-category differential output rather
  than a discrete hit list that can be tabulated row-wise.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: Python
  role: Custom scripts assemble the per-compound result table and write the CSV output
- name: ClassyFire
  role: Provides the chemical-ontology superclass and class fields populated into
    the assembled table
- name: BinDiscover
  role: Source of the significant-metabolite list (OGDA query results) that is tabulated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:f935a5d63b683e53da39751f1e657f5876706e75e59c922add1c77090c1f8111
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinDiscover
  - ClassyFire
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tabular-result-assembly

## Summary

Assemble statistical result outputs into structured CSV tables that pair each compound or category with its metadata annotations, so that downstream chemical-ontology or pathway queries can consume the hit list directly.

## When to use

Use when a BinDiscover OGDA query yields a list of significant metabolites (e.g. 15 hits from a human-digestive-tract-vs-bacteria comparison) that must be exported with per-compound annotation columns for downstream enrichment.

## When NOT to use

- Input is already a fully annotated feature table with non-empty ontology columns — assembling it again adds no information.
- The OGDA query returned a graph-based or single-category differential output rather than a discrete hit list that can be tabulated row-wise.

## Inputs

- List of significant metabolites from a BinDiscover OGDA query (e.g. 15 hits from Additional file 3: Table S2)
- Per-compound annotation fields (e.g. ClassyFire superclass and class, or ontology-mapped BinBase metadata)

## Outputs

- CSV table with one row per compound and columns for compound name plus annotation fields (e.g. classyfire_classification.csv)

## How to apply

Collect the significant metabolites produced by an OGDA query — for example the 15 hits in Additional file 3: Table S2 from the human-digestive-tract-vs-bacteria showcase — and build a per-compound table with one row per hit. Each row carries the compound name plus annotation columns such as ClassyFire superclass and class, so that all 15 metabolites carry non-empty chemical-ontology assignments consistent with the paper's summary categories (vitamins, lipids, sterols, amino acid derivatives). Write the table to a CSV (e.g. classyfire_classification.csv) and verify that no row has an empty class field. The underlying metadata for these rows derives from BinBase's 18,290 Bins (773 identified metabolites) whose identifiers were mapped to established ontologies and corrected for misspellings before assembly.

## Related tools

- **Python** (Custom scripts assemble the per-compound result table and write the CSV output)
- **ClassyFire** (Provides the chemical-ontology superclass and class fields populated into the assembled table)
- **BinDiscover** (Source of the significant-metabolite list (OGDA query results) that is tabulated)

## Examples

```
python scripts/assemble_results.py --input ogda_hits.tsv --output classyfire_classification.csv
```

## Evaluation signals

- All 15 metabolites in the assembled CSV carry non-empty ClassyFire superclass and class values.
- Class assignments are consistent with the paper's summary categories (vitamins, lipids, sterols, amino acid derivatives).
- Row count of the CSV equals the number of significant hits in the source OGDA result (e.g. 15).
- Each compound name in the table matches a BinBase-identified metabolite from the 773 identified metabolites.
- CSV parses without errors and column headers match the expected schema (compound name, superclass, class).

## Limitations

- BinDiscover relies on a static December 2021 snapshot of BinBase, so novel compounds added later may lack ontology assignments in the assembled table.
- For biological metadata combinations with few samples and few studies, the assembled quantitative columns are less robust.
- Cross-organ comparisons may mix biofluids versus tissues (different units of biomass), which affects the interpretation of assembled intensity values.

## Evidence

- [supplementary] List of 15 significant metabolites from the human-digestive-tract-vs-bacteria OGDA query results (Additional file 3: Table S2).: [evidence span withheld — anchor status: not_found]
- [supplementary] Write the table to classyfire_classification.csv and verify that all 15 metabolites carry non-empty class assignments consistent with the paper's summary categories (vitamins, lipids, sterols, and amino acid derivatives).: [evidence span withheld — anchor status: not_found]
- [intro] OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs.: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [methods] This data comprised 156,174 samples that were processed into 18,290 Bins: "This data comprised 156,174 samples that were processed into 18,290 Bins"
- [methods] Bins included 773 identified metabolites: "Bins included 773 identified metabolites"
- [methods] we curated and combined metadata combinations to map metadata to established ontologies and to correct for misspellings: "we curated and combined metadata combinations to map metadata to established ontologies and to correct for misspellings"
