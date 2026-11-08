---
name: species-coverage-verification
description: Use when building or auditing the scope statement of a multiomics resource
  and you need to confirm that the listed species, tissue subtypes, cell lines, and
  gene counts match the values reported in the source article or repository metadata.
when_to_use_negative:
- When the input is already a validated coverage table or a machine-readable inventory
  that needs no re-verification.
- When the goal is to explore or query the database’s functional modules (Browse,
  Query, Visualization, Download) rather than verify its declared scope figures.
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used to parse the metadata, cross-check declared values, and write the coverage
    inventory CSV.
  repo: https://github.com/Ansanqi/GutUDB
- name: pandas
  role: Used to construct the inventory table and export it to CSV.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:c6e0911b3266235022071c635de945d43cfef571e9aba5156e7204628c20b1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# species-coverage-verification

## Summary

Verify and tabulate the species, tissue-subtype, cell-line, and gene-count coverage of a multiomics database such as GutUDB, ensuring the declared inventory (six species, 11 intestinal tissue subtypes, 63 intestinal cell lines, 58,970 genes) is accurately recorded.

## When to use

Use when building or auditing the scope statement of a multiomics resource and you need to confirm that the listed species, tissue subtypes, cell lines, and gene counts match the values reported in the source article or repository metadata.

## When NOT to use

- When the input is already a validated coverage table or a machine-readable inventory that needs no re-verification.
- When the goal is to explore or query the database’s functional modules (Browse, Query, Visualization, Download) rather than verify its declared scope figures.

## Inputs

- GutUDB source metadata (article abstract or repository README) containing declared species, tissue-subtype, cell-line, and gene-count figures
- Reference figures: six species, 11 intestinal tissue subtypes, 63 intestinal cell lines, 58,970 genes

## Outputs

- coverage_inventory.csv — a CSV table with one row per coverage dimension (species, tissue subtypes, cell lines, genes), showing declared and reference values

## How to apply

Parse the database’s source metadata (abstract or repository README) to extract the declared species list, the number of intestinal tissue subtypes, the number of intestinal cell lines, and the total gene count. Cross-check each extracted value against the explicitly reported figures—six species, 11 tissue subtypes, 63 cell lines, and 58,970 genes—flagging any mismatch. Assemble a machine-readable coverage inventory with one row per dimension (species, tissue subtypes, cell lines, genes) using pandas, recording both the declared and the reference value. Write the verified table to a CSV file (coverage_inventory.csv) and validate that the output exists, parses correctly, and matches the reference figures.

## Related tools

- **Python** (Used to parse the metadata, cross-check declared values, and write the coverage inventory CSV.) — https://github.com/Ansanqi/GutUDB
- **pandas** (Used to construct the inventory table and export it to CSV.)

## Examples

```
import pandas as pd
coverage = pd.DataFrame({'dimension': ['species', 'tissue_subtypes', 'cell_lines', 'genes'],
                         'declared': [6, 11, 63, 58970],
                         'reference': [6, 11, 63, 58970]})
coverage.to_csv('coverage_inventory.csv', index=False)
```

## Evaluation signals

- The output CSV exists and parses without errors.
- The recorded values for species (6), tissue subtypes (11), cell lines (63), and genes (58,970) exactly match the reference figures.
- Each coverage dimension appears as a separate row in the inventory.
- No mismatch flags are raised when declared values are compared to reference values.
- The CSV includes both declared and reference columns for each dimension.

## Limitations

- The verification depends on the accuracy of the declared figures in the abstract or README; if the source metadata is outdated or incomplete, the check will pass or fail incorrectly.
- The skill does not validate the actual underlying data (e.g., that the database truly contains data for all six species); it only reconciles reported numbers.
- Counts may change with future database updates, requiring the reference figures to be re-sourced.

## Evidence

- [abstract] GutUDB covers six species and derived 58,970 genes from 11 subtypes of intestinal tissue or 63 intestinal cell lines: "GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines, and identified various potential clinical therapeutics"
- [abstract] GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets covering 56 intestinal diseases across six species: "To date, GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
