---
name: categorical-frequency-counting
description: Use when a labeled spreadsheet/CSV has one categorical column (e.g.,
  peptide hormone family annotations) and the goal is a frequency table of all distinct
  categories plus identification of the most abundant category.
when_to_use_negative:
- The input lacks a harmonized categorical label column — entries must first receive
  exactly one family label under the UniProt-based criteria before counting.
- The goal is continuous-variable statistics or correlation analysis rather than discrete
  category frequencies.
- The entity set is not the HORDB peptide hormone dataset and no comparable curated
  classification scheme exists for it.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: read_excel to load the HORDB xlsx and value_counts to compute per-family frequencies
- name: matplotlib
  role: render the family-size bar plot corresponding to Fig. 8
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:eec009bba8c7782a8d811636bc699ffeccff082631f93d6b8a643f452964b743
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# categorical-frequency-counting

## Summary

Counts and ranks category frequencies over an annotated spreadsheet of entities — here, the HORDB 'Peptide hormones infomation.xlsx' file classified into UniProt-based peptide hormone families — to reproduce distribution tables and bar plots such as HORDB's Fig. 8 (80 families, FMRFamide-related peptides most abundant with 856 entries).

## When to use

Use when a labeled spreadsheet/CSV has one categorical column (e.g., peptide hormone family annotations) and the goal is a frequency table of all distinct categories plus identification of the most abundant category.

## When NOT to use

- The input lacks a harmonized categorical label column — entries must first receive exactly one family label under the UniProt-based criteria before counting.
- The goal is continuous-variable statistics or correlation analysis rather than discrete category frequencies.
- The entity set is not the HORDB peptide hormone dataset and no comparable curated classification scheme exists for it.

## Inputs

- Excel spreadsheet of annotated entities ('Peptide hormones infomation.xlsx') with a family/classification column
- UniProt-based family annotation criteria for label assignment and synonym harmonization

## Outputs

- peptide_hormone_family_counts.csv — family-to-count frequency table covering all 80 distinct families
- Family-size bar plot (matplotlib) reproducing the Fig. 8 distribution

## How to apply

Load the deposited spreadsheet ('Peptide hormones infomation.xlsx' from the HORDB figshare collection, https://doi.org/10.6084/m9.figshare.c.5522241) with pandas.read_excel. Assign each peptide hormone entry exactly one family label using the UniProt-based protein family classification criteria, harmonizing synonym labels so no entry is double-counted or left unlabeled. Apply pandas value_counts to the family column, keep the full set of distinct categories (80 for HORDB), sort counts in descending order, and verify the top category — FMRFamide-related peptides with 856 members — before writing the table to CSV and rendering a bar plot with matplotlib as in Fig. 8.

## Related tools

- **pandas** (read_excel to load the HORDB xlsx and value_counts to compute per-family frequencies)
- **matplotlib** (render the family-size bar plot corresponding to Fig. 8)

## Examples

```
import pandas as pd; df = pd.read_excel('Peptide hormones infomation.xlsx'); counts = df['family'].value_counts(); counts.to_csv('peptide_hormone_family_counts.csv')
```

## Evaluation signals

- The value_counts output contains exactly 80 distinct family categories.
- The top-ranked family is FMRFamide-related peptides with a count of 856.
- Frequencies sum to the total number of entries in the spreadsheet (no dropped or duplicated rows after synonym harmonization).
- The written CSV rows match the descending-sorted counts used to render the Fig. 8 bar plot.
- The distribution matches the reported Fig. 8 family ranking.

## Limitations

- Counts depend on the synonym-harmonization step: inconsistent family labels inflate the number of distinct categories and split member counts.
- The 80-family / 856-member result reflects the current HORDB snapshot (6024 entries); future figshare deposits may change the distribution.
- Each entry must map to exactly one family; entries with ambiguous or multiple UniProt family annotations require a decision rule before counting.

## Evidence

- [results] Under the UniProt-based protein classification criteria, HORDB contains 80 families, and the most abundant family is the FMRFamide related peptide family with 856 members in total.: [evidence span withheld — anchor status: not_found]
- [abstract] All data are hosted and available in figshare: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
