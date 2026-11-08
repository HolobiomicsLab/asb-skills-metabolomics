---
name: tabular-data-summarization
description: Use when a spreadsheet or table of entities (e.g. peptide hormone entries
  with family annotations) must be summarized into a frequency table per category,
  sorted, validated against reported counts, and plotted.
when_to_use_negative:
- The table lacks a consistent categorical annotation column, so counts would not
  be meaningful without prior harmonization of synonym labels.
- The goal is database record-level querying of HORDB's 6024 entries rather than aggregating
  entries into a category frequency summary.
- Categories are continuous numeric values requiring binning/density estimation instead
  of exact categorical value_counts.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: read_excel to load the HORDB xlsx and value_counts/sort to compute and rank
    the 80 family frequencies; to_csv to export the table
  repo: CPU-HORDB/HORDB
- name: matplotlib
  role: render the family-size bar plot reproducing the Fig. 8 distribution of HORDB
    families
  repo: CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:a39910755781190e193762064342c9b58895d5998e290562a1b4c65947c4d3d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CPU-HORDB/HORDB
  tools:
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# tabular-data-summarization

## Summary

Summarize a tabular dataset by computing categorical frequency counts and visualizing their distribution — as done for HORDB, where the 'Peptide hormones infomation.xlsx' spreadsheet was aggregated into an 80-family table (Fig. 8) showing FMRFamide-related peptides as the most abundant family with 856 members.

## When to use

Use when a spreadsheet or table of entities (e.g. peptide hormone entries with family annotations) must be summarized into a frequency table per category, sorted, validated against reported counts, and plotted.

## When NOT to use

- The table lacks a consistent categorical annotation column, so counts would not be meaningful without prior harmonization of synonym labels.
- The goal is database record-level querying of HORDB's 6024 entries rather than aggregating entries into a category frequency summary.
- Categories are continuous numeric values requiring binning/density estimation instead of exact categorical value_counts.

## Inputs

- Excel spreadsheet 'Peptide hormones infomation.xlsx' (HORDB figshare collection https://doi.org/10.6084/m9.figshare.c.5522241)
- Column of categorical family annotations (UniProt-based family labels, one per peptide hormone entry)

## Outputs

- Family-to-count frequency table (80 families) written to peptide_hormone_family_counts.csv
- Family-size bar plot (as in Fig. 8) rendered with matplotlib

## How to apply

Load the source spreadsheet (e.g. HORDB's 'Peptide hormones infomation.xlsx' from the figshare collection 10.6084/m9.figshare.c.5522241) with pandas.read_excel. Assign each entry exactly one category label (here, UniProt-based family annotation with synonym labels harmonized). Apply pandas value_counts to the category column, keep all distinct categories (HORDB: 80 families), sort counts descending, and write the result to CSV (peptide_hormone_family_counts.csv). Render the category-size distribution as a bar plot with matplotlib, mirroring Fig. 8 of the HORDB paper. Validate that the number of distinct categories and the top-ranked count reproduce the reported values (80 families; FMRFamide-related peptides, 856 members).

## Related tools

- **pandas** (read_excel to load the HORDB xlsx and value_counts/sort to compute and rank the 80 family frequencies; to_csv to export the table) — CPU-HORDB/HORDB
- **matplotlib** (render the family-size bar plot reproducing the Fig. 8 distribution of HORDB families) — CPU-HORDB/HORDB

## Examples

```
import pandas; counts = pandas.read_excel('Peptide hormones infomation.xlsx')['family'].value_counts().sort_values(ascending=False); counts.to_csv('peptide_hormone_family_counts.csv')
```

## Evaluation signals

- The output table contains exactly 80 distinct family categories.
- The top-ranked family is FMRFamide-related peptides with a count of 856.
- Sum of all family counts equals the total number of classified peptide hormone entries (each entry receives exactly one label).
- Exported CSV matches the rendered bar plot ordering and heights (as in Fig. 8).
- value_counts output is sorted in strictly descending order with no NaN category label.

## Limitations

- Synonym family labels must be harmonized first; otherwise distinct labels inflate the family count beyond 80.
- The reported totals reflect the current HORDB release (6024 entries including 5729 peptide hormones); updates to the figshare deposit will change counts.
- Classification depends on the UniProt-based protein classification criteria used by HORDB; other classification schemes will yield different family tables.

## Evidence

- [abstract] The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information.: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information."
- [abstract] All data are hosted and available in figshare: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
