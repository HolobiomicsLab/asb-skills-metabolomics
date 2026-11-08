---
name: csv-table-consolidation
description: Use when a metabolomics annotation table (e.g. REname output CSV) contains
  many redundant putative annotation rows per feature and the goal is a size-reduced
  table for manual comparison of annotations.
when_to_use_negative:
- The table needs compound classification or biological-system insight rather than
  row reduction — Tagger, not RowMerger, is the module for that
- The dataset pairs feature intensities and metadata and needs an adduct-probability/correlation
  score — TPMetrics, not RowMerger, handles that
- Input is already a consolidated feature table with one annotation row per mass/adduct/error
  group, so no redundant rows remain to merge
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Its RowMerger module performs the grouping of shared-mass/adduct/mass-error
    annotation rows and the '//' name concatenation; runnable via the web server at
    https://proteomics.cnic.es/TurboPutative/ or its RESTful API
  repo: CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:5feeb052afaddee57d20d895a666fb002a6b960729cdb33cb6b62265d29024f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - TurboPutative
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# csv-table-consolidation

## Summary

Consolidates a putative annotation table by merging rows that describe the same metabolite feature, using TurboPutative's RowMerger logic to collapse redundant annotation entries so downstream manual comparison becomes tractable.

## When to use

Use when a metabolomics annotation table (e.g. REname output CSV) contains many redundant putative annotation rows per feature and the goal is a size-reduced table for manual comparison of annotations.

## When NOT to use

- The table needs compound classification or biological-system insight rather than row reduction — Tagger, not RowMerger, is the module for that
- The dataset pairs feature intensities and metadata and needs an adduct-probability/correlation score — TPMetrics, not RowMerger, handles that
- Input is already a consolidated feature table with one annotation row per mass/adduct/error group, so no redundant rows remain to merge

## Inputs

- Abbreviated putative annotation table from the REname step (CSV, one row per potential annotation with columns for experimental neutral mass, m/z, adduct, and mass error)

## Outputs

- Consolidated annotation table (rowmerger_output.csv) with redundant rows collapsed and metabolite names joined by the '//' delimiter

## How to apply

Load the abbreviated annotation table produced by the REname step (CSV) with pandas, then apply RowMerger's grouping logic: group entries that share identical experimental neutral mass, m/z, adduct, and mass error (the default criteria), while keeping the user-selectable comparison criteria configurable. For each merged group, concatenate the abbreviated metabolite names into a single cell separated by the '//' delimiter, so multiple potential annotations collapse into one row. Write the consolidated table to rowmerger_output.csv. The rationale is that RowMerger 'reduces the dataset size, facilitating the manual comparison among annotations'; note that the abbreviated name itself is not used as a grouping key — names are instead concatenated with '//' once rows match on the mass/adduct/error criteria.

## Related tools

- **TurboPutative** (Its RowMerger module performs the grouping of shared-mass/adduct/mass-error annotation rows and the '//' name concatenation; runnable via the web server at https://proteomics.cnic.es/TurboPutative/ or its RESTful API) — CNIC-Proteomics/TurboPutative-web

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('rename_output.csv'); df.to_csv('rowmerger_output.csv')"  # group rows sharing experimental mass, m/z, adduct, mass error; join names with '//'
```

## Evaluation signals

- Merged name cells contain the '//' separator joining previously separate abbreviated metabolite names
- Row count of rowmerger_output.csv is strictly less than the input REname table; in massive datasets expect roughly 80%-90% reduction of putative annotation entries
- All rows within each output group share identical experimental neutral mass, m/z, adduct, and mass error values
- No abbreviated name was used as a grouping criterion — distinct metabolite names appear concatenated within a single merged row rather than split across rows
- Output CSV parses cleanly in pandas with the same annotation columns as the input table

## Limitations

- Default grouping ignores the abbreviated metabolite name, so metabolically distinct isomers/equivalent compounds sharing mass, adduct, and mass error are merged into one '//'-delimited cell and remain indistinguishable at this stage
- Grouping criteria are user-customizable, so non-default configurations change which rows collapse and may not reproduce the default mass/adduct/error behavior
- RowMerger only reduces table size for comparison; it does not classify compounds (Tagger) or score features by adduct probability and correlations (TPMetrics)

## Evidence

- [abstract] RowMerger reduces dataset size to facilitate manual comparison among annotations: "RowMerger reduces the dataset size, facilitating the manual comparison among annotations"
- [abstract] TurboPutative reduces putative annotation entries by 80%-90% in massive datasets: "dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] TurboPutative comprises four modules including RowMerger: "a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
