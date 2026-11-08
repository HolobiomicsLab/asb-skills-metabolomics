---
name: category-level-record-classification-verification
description: Use when auditing a curated database deposit (e.g., HORDB's /datasets/
  folder) to confirm per-category record counts match the manuscript-reported totals,
  such as 6024 total entries split into 5729/40/255.
when_to_use_negative:
- The deposit is not a categorically structured database (no per-category entry tables)
  and count reconciliation against a manuscript total is not the goal
- You need biological/sequence validation of HORDB records (e.g., IC50/EC50/ED50 activity
  fields or sequence checks) rather than record-count classification verification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Python (pandas)
  role: Load the per-category HORDB entry tables and count unique entries via value
    counting on the entry identifier column
- name: HORDB
  role: Source database whose /datasets/ folder holds all per-category datasets being
    verified
  repo: https://github.com/CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:9ccf490aa21688c9fd2c6cd182b878e0fcb1003dbf61e5eb49f9fc2ec9542134
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CPU-HORDB/HORDB
  tools:
  - HORDB
  - Python
  - Python (pandas)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# category-level-record-classification-verification

## Summary

Verifies that records deposited in the HORDB figshare collection (DOI 10.6084/m9.figshare.c.5522241) classify into the three HORDB categories — peptide hormones, peptide drugs, and marketed pharmaceutical preparations — and that category counts reconcile to the published totals of 6024 entries.

## When to use

Use when auditing a curated database deposit (e.g., HORDB's /datasets/ folder) to confirm per-category record counts match the manuscript-reported totals, such as 6024 total entries split into 5729/40/255.

## When NOT to use

- The deposit is not a categorically structured database (no per-category entry tables) and count reconciliation against a manuscript total is not the goal
- You need biological/sequence validation of HORDB records (e.g., IC50/EC50/ED50 activity fields or sequence checks) rather than record-count classification verification

## Inputs

- figshare dataset files from DOI 10.6084/m9.figshare.c.5522241 (or the repository /datasets/ folder)
- Per-category HORDB entry tables (peptide hormones, peptide drugs, marketed pharmaceutical preparations)
- Published count expectations from the Data Records section (6024 total; 5729/40/255 breakdown)

## Outputs

- entry_count_breakdown.csv containing the total entry count plus the three category counts
- Per-category unique-entry counts reconciled against reported figures

## How to apply

Download all dataset files from the figshare collection (DOI 10.6084/m9.figshare.c.5522241) or clone the CPU-HORDB/HORDB repository, whose /datasets/ folder 'Contains all datasets from HORDB'. Load the per-category entry tables with Python (pandas), identifying which files correspond to peptide hormones, peptide drugs, and marketed pharmaceutical preparations. Count unique entries per category file using pandas value counting on the entry identifier column to avoid double-counting repeated identifiers. Sum the three category counts and compare against the Data Records totals: 5729 peptide hormones + 40 peptide drugs + 255 marketed pharmaceutical preparations = 6024 total entries. Any mismatch signals misclassified, duplicated, or missing records in the deposit.

## Related tools

- **Python (pandas)** (Load the per-category HORDB entry tables and count unique entries via value counting on the entry identifier column)
- **HORDB** (Source database whose /datasets/ folder holds all per-category datasets being verified) — https://github.com/CPU-HORDB/HORDB

## Examples

```
df = pd.read_csv('datasets/peptide_hormones.csv'); df['entry_id'].value_counts().shape[0]  # expect 5729; sum all categories to verify 6024
```

## Evaluation signals

- Recomputed total equals 6024 entries exactly
- Category breakdown matches 5729 peptide hormones, 40 peptide drugs, 255 marketed pharmaceutical preparations
- Sum of the three category counts equals the recomputed total (5729 + 40 + 255 = 6024)
- entry_count_breakdown.csv is written with the total and all three category counts
- Counting used unique entry identifiers (no duplicate entries inflated counts)

## Limitations

- Verification depends on the figshare deposit (10.6084/m9.figshare.c.5522241) matching the manuscript's Data Records totals; updates to the deposit will break the 6024/5729/40/255 invariant
- Entry identifier columns must be unambiguous — duplicate or missing identifiers make value counting unreliable
- Count reconciliation does not validate record content (sequences, activity data such as IC50/EC50/ED50, targets, or blood-brain barrier annotations)
