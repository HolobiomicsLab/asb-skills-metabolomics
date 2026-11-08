---
name: peptide-activity-field-filtering
description: Use when you have a tabular HORDB peptide hormone information table (from
  the figshare deposit 10.6084/m9.figshare.c.5522241) and need to filter rows by an
  exact string in the activity field to validate a reported entry count.
when_to_use_negative:
- Input is not the HORDB peptide hormone information table (e.g. peptide drug or pharmaceutical
  preparation tables from the 6024-entry database), since the 'cross BBB' annotation
  applies to the peptide hormone activity field.
- The analysis goal is fuzzy or semantic activity matching (e.g. NLP-based extraction
  of BBB permeability) rather than an exact string match on the deposited activity
  field.
- You need computed physicochemical predictions of BBB crossing rather than the recorded
  activity annotations; this skill only validates what is deposited.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3071
tools:
- name: pandas
  role: Loads the HORDB peptide hormone information table, performs the exact string-match
    filter on the activity column, writes bbb_crossing_peptides.csv, and computes
    the row count.
- name: HORDB
  role: Source database whose peptide hormone information table (activity, sequence,
    structure, physical and literature information) is downloaded from figshare and
    filtered.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:9ba4388c15fdf67382394c71c5de66bfe53f90cd59e9f839e1f5bfddeb25fac4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HORDB
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# peptide-activity-field-filtering

## Summary

A pandas-based filtering skill for the HORDB peptide hormone information table that counts and extracts entries carrying a specific activity annotation (e.g. 'cross BBB') to verify reported database statistics.

## When to use

Use when you have a tabular HORDB peptide hormone information table (from the figshare deposit 10.6084/m9.figshare.c.5522241) and need to filter rows by an exact string in the activity field to validate a reported entry count.

## When NOT to use

- Input is not the HORDB peptide hormone information table (e.g. peptide drug or pharmaceutical preparation tables from the 6024-entry database), since the 'cross BBB' annotation applies to the peptide hormone activity field.
- The analysis goal is fuzzy or semantic activity matching (e.g. NLP-based extraction of BBB permeability) rather than an exact string match on the deposited activity field.
- You need computed physicochemical predictions of BBB crossing rather than the recorded activity annotations; this skill only validates what is deposited.

## Inputs

- HORDB peptide hormone information table (tabular file) from the figshare deposit https://doi.org/10.6084/m9.figshare.c.5522241
- Activity column containing annotations such as 'cross BBB', IC50, EC50, ED50, and target information

## Outputs

- bbb_crossing_peptides.csv — filtered subset of peptide hormone entries whose activity field contains the 'cross BBB' annotation
- Row count of the filtered subset, compared against the reported value of 28

## How to apply

Download the HORDB peptide hormone information table from the figshare collection at https://doi.org/10.6084/m9.figshare.c.5522241 and load it with pandas. Apply an exact string match on the activity column — for blood-brain barrier crossing, filter rows whose activity field contains the 'cross BBB' annotation. Write the filtered subset to bbb_crossing_peptides.csv and compute its row count with pandas. Compare the computed row count against the reported value of 28 peptide hormones that cross the blood-brain barrier, as stated in the HORDB database statistics. The rationale is that HORDB annotates each entry's activity notes with quantitative and qualitative measures including IC50, EC50, ED50, target, and blood-brain barrier crossing information, so string filtering on this column reconstructs the statistics directly from the deposited data.

## Related tools

- **pandas** (Loads the HORDB peptide hormone information table, performs the exact string-match filter on the activity column, writes bbb_crossing_peptides.csv, and computes the row count.)
- **HORDB** (Source database whose peptide hormone information table (activity, sequence, structure, physical and literature information) is downloaded from figshare and filtered.)

## Examples

```
import pandas as pd; df = pd.read_csv('hordb_peptide_hormones.csv'); bbb = df[df['activity'].str.contains('cross BBB', regex=False)]; bbb.to_csv('bbb_crossing_peptides.csv', index=False); print(len(bbb))
```

## Evaluation signals

- The filtered subset written to bbb_crossing_peptides.csv contains exactly 28 rows, matching the HORDB-reported count of peptide hormones that cross the blood-brain barrier.
- Every row in the filtered subset has the exact 'cross BBB' string in its activity field.
- The filter is applied to the activity column only — entries lacking this annotation (among the 5729 peptide hormones) must not appear in the output.
- The output CSV preserves all columns of the original HORDB table for the matched rows (sequence, activity, structure, physical and literature information).

## Limitations

- Count depends on the exact string convention in the deposited activity field; partial or differently formatted annotations would change the count away from the reported 28.
- The check validates only what HORDB records — it does not independently confirm blood-brain barrier permeability of the peptide hormones.
- Requires access to the figshare deposit (10.6084/m9.figshare.c.5522241); if the deposit or HORDB download page (http://hordb.cpu-bioinfor.org/) is unavailable, the filter cannot be run.

## Evidence

- [other] Database statistics / reported count: [evidence span withheld — anchor status: not_found]
- [abstract] Entry content / activity annotations: "We also added information on IC50, EC50, ED50, target, and whether or not the blood-brain barrier was crossed to the activity information note."
- [abstract] Data availability: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [abstract] Database scale: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information."
- [readme] Dataset locations: "/datasets/ Contains all datasets from HORDB"
