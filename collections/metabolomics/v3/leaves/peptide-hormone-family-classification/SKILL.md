---
name: peptide-hormone-family-classification
description: Use when the input is a peptide-hormone table (e.g. HORDB's xlsx) needing
  family assignment under UniProt protein classification criteria, with per-family
  frequency counts and a family-size distribution plot.
when_to_use_negative:
- Input data are non-hormone DRAMP antimicrobial peptides or HORDB peptide drugs rather
  than entries that map to the peptide-hormone family taxonomy.
- The goal is sequence-level de novo family discovery rather than assigning labels
  via existing UniProt-based classification criteria.
- Only a subset of HORDB families is of interest and a partial distribution is acceptable
  — this skill requires retaining the full set of 80 distinct families.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the HORDB xlsx spreadsheet with read_excel and computes family frequencies
    with value_counts
- name: matplotlib
  role: Renders the family-size bar plot reproducing Fig. 8
- name: HORDB source code (CPU-HORDB/HORDB)
  role: Repository providing the code to build HORDB and its datasets, including the
    family classification pipeline
  repo: CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:abe0dfb7f0740ded20869860d215a2de6b8432b6b209e0215038cbc145f8bee7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CPU-HORDB/HORDB
  tools:
  - HORDB source code (CPU-HORDB/HORDB)
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# peptide-hormone-family-classification

## Summary

Classifies the 6024 HORDB peptide hormone entries into UniProt-based protein families using the deposited 'Peptide hormones infomation.xlsx' spreadsheet, then tabulates and plots family-size counts. It reproduces the paper's finding of 80 families with FMRFamide-related peptides as the largest (856 members), mirroring Fig. 8.

## When to use

Use when the input is a peptide-hormone table (e.g. HORDB's xlsx) needing family assignment under UniProt protein classification criteria, with per-family frequency counts and a family-size distribution plot.

## When NOT to use

- Input data are non-hormone DRAMP antimicrobial peptides or HORDB peptide drugs rather than entries that map to the peptide-hormone family taxonomy.
- The goal is sequence-level de novo family discovery rather than assigning labels via existing UniProt-based classification criteria.
- Only a subset of HORDB families is of interest and a partial distribution is acceptable — this skill requires retaining the full set of 80 distinct families.

## Inputs

- Peptide hormones infomation.xlsx spreadsheet from the HORDB figshare collection (https://doi.org/10.6084/m9.figshare.c.5522241)
- UniProt-based protein family classification criteria/annotation mappings

## Outputs

- peptide_hormone_family_counts.csv — family-to-count table covering 80 family categories
- Matplotlib family-size bar plot reproducing the Fig. 8 distribution

## How to apply

Load 'Peptide hormones infomation.xlsx' from the HORDB figshare collection (doi:10.6084/m9.figshare.c.5522241) with pandas.read_excel. Assign each peptide hormone entry exactly one family label per the UniProt-based protein classification criteria, harmonizing synonym labels so no entry lands in two families. Compute per-family frequencies with pandas value_counts, keeping all 80 distinct family categories, then sort descending. Verify the top family is FMRFamide-related peptides with 856 members and that the number of distinct families equals 80. Export the family-to-count table to CSV and render a bar plot (as in Fig. 8) with matplotlib.

## Related tools

- **pandas** (Loads the HORDB xlsx spreadsheet with read_excel and computes family frequencies with value_counts)
- **matplotlib** (Renders the family-size bar plot reproducing Fig. 8)
- **HORDB source code (CPU-HORDB/HORDB)** (Repository providing the code to build HORDB and its datasets, including the family classification pipeline) — CPU-HORDB/HORDB

## Examples

```
import pandas as pd, matplotlib.pyplot as plt; df = pd.read_excel('Peptide hormones infomation.xlsx'); counts = df['family'].value_counts(); counts.to_csv('peptide_hormone_family_counts.csv'); counts.plot.bar()
```

## Evaluation signals

- The family count table contains exactly 80 distinct family categories.
- The top-ranked family is FMRFamide-related peptides with a count of 856.
- Total classified entries reconcile with HORDB's 5729 peptide hormones (6024 total entries minus 40 peptide drugs and 255 pharmaceutical preparations).
- Every entry receives exactly one family label after synonym harmonization (no unassigned or double-assigned rows).
- The rendered bar plot's rank ordering matches the reported Fig. 8 distribution.

## Limitations

- Counts depend on harmonizing synonym family labels correctly; inconsistent synonym merging will change the 80-family and 856-member totals.
- Results are tied to the deposited figshare version of HORDB (6024 entries); future database updates may shift family counts.
- The README notes datasets are also hosted at http://hordb.cpu-bioinfor.org/ — a different download source must match the figshare snapshot to reproduce the validation.

## Evidence

- [abstract] The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information.: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information."
- [abstract] All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241"
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
