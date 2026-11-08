---
name: blood-brain-barrier-annotation-extraction
description: Use when a HORDB peptide hormone table (figshare deposit 10.6084/m9.figshare.c.5522241)
  needs filtering on the 'cross BBB' activity annotation to quantify BBB-crossing
  peptide hormones.
when_to_use_negative:
- The table is not the HORDB peptide hormone information table (e.g. peptide drugs
  or marketed pharmaceutical preparations), since the 'cross BBB' field is defined
  for peptide hormone entries.
- A quantitative BBB permeability prediction is needed — this skill only extracts
  the deposited annotation, not a computed permeability value.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3062
tools:
- name: pandas
  role: Loads the HORDB peptide hormone table and filters rows on the 'cross BBB'
    activity annotation
- name: HORDB
  role: Source database whose peptide hormone entries carry the blood-brain barrier
    crossing annotation in the activity field
  repo: CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:81c13d025cdbbd47a66ffbe9c7643ca8acebaaa455337000e39dbf40ef7d2242
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CPU-HORDB/HORDB
  tools:
  - HORDB
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3062
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# blood-brain-barrier-annotation-extraction

## Summary

Extracts and counts peptide hormone entries from the HORDB database that are annotated as crossing the blood-brain barrier, by exact string filtering on the activity field of the deposited peptide information table.

## When to use

Use when a HORDB peptide hormone table (figshare deposit 10.6084/m9.figshare.c.5522241) needs filtering on the 'cross BBB' activity annotation to quantify BBB-crossing peptide hormones.

## When NOT to use

- The table is not the HORDB peptide hormone information table (e.g. peptide drugs or marketed pharmaceutical preparations), since the 'cross BBB' field is defined for peptide hormone entries.
- A quantitative BBB permeability prediction is needed — this skill only extracts the deposited annotation, not a computed permeability value.

## Inputs

- HORDB peptide hormone information table (downloaded from figshare deposit https://doi.org/10.6084/m9.figshare.c.5522241)
- Activity column containing 'cross BBB' annotation strings

## Outputs

- bbb_crossing_peptides.csv — filtered subset of peptide hormones whose activity field contains 'cross BBB'
- Row count of the filtered subset for comparison against the reported value of 28

## How to apply

Download the HORDB peptide hormone information table from the figshare deposit (https://doi.org/10.6084/m9.figshare.c.5522241) and load it with pandas. Filter rows whose activity column contains the 'cross BBB' annotation using an exact string match on that field, since HORDB records blood-brain barrier crossing information within the activity information notes. Write the filtered subset to bbb_crossing_peptides.csv and compute its row count. Validate by comparing the computed count against the reported 28 peptide hormones recorded as crossing the blood-brain barrier; a mismatch indicates a string-matching or table-versioning error.

## Related tools

- **pandas** (Loads the HORDB peptide hormone table and filters rows on the 'cross BBB' activity annotation)
- **HORDB** (Source database whose peptide hormone entries carry the blood-brain barrier crossing annotation in the activity field) — CPU-HORDB/HORDB

## Examples

```
import pandas as pd; df = pd.read_csv('hordb_peptides.csv'); sub = df[df['activity'].str.contains('cross BBB', regex=False)]; sub.to_csv('bbb_crossing_peptides.csv', index=False); len(sub)
```

## Evaluation signals

- The filtered subset contains exactly 28 rows, matching the reported count of peptide hormones that cross the blood-brain barrier.
- Every row in bbb_crossing_peptides.csv has an activity field containing the exact string 'cross BBB'.
- The filtered row count is <= the total of 5729 peptide hormone entries in HORDB.
- Re-running the pandas filter on the same downloaded table reproduces an identical subset.

## Limitations

- Exact string matching on the activity column is brittle to annotation format variation (e.g. capitalization or extra whitespace), which would change the count away from 28.
- The verification depends on the current figshare deposit; updates to the HORDB datasets (5729 peptide hormones at time of publication) would invalidate the 28-entry expectation.

## Evidence

- [abstract] cross BBB annotation count: "We also added information on IC50, EC50, ED50, target, and whether or not the blood-brain barrier was crossed to the activity information note."
- [abstract] database scale: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information."
- [abstract] data hosting: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] dataset location: "/datasets/ Contains all datasets from HORDB"
