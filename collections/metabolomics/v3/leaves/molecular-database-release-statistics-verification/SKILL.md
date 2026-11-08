---
name: molecular-database-release-statistics-verification
description: Use when a database release report (e.g. COCONUT October 2020) states
  aggregate counts (unique flat NPs, stereo molecules) and per-source tables, and
  those numbers must be independently recomputed and reconciled.
when_to_use_negative:
- The database already publishes machine-readable per-source and per-release count
  dumps that you can load directly, making manual reconciliation redundant.
- You need per-molecule provenance or structural deduplication logic rather than only
  aggregate release-level count verification.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2258
tools:
- name: pandas
  role: Used to load the per-source entry counts from Table 1, assemble them into
    a single per-source counts table, and compute the column total.
- name: MongoDB
  role: Serves as the target store where the assembled per-source counts table and
    verification summary can be written for cross-checking against the database's
    unified entries.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:0260b31e594ff89f016a010e5a85f306ec9324fd56bbca849bb81bc2623bbbe9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MongoDB
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# molecular-database-release-statistics-verification

## Summary

Verifies the aggregate molecule counts of a release of a curated molecular database (e.g. COCONUT) by recomputing per-source entry totals and reconciling them against the reported unique flat (stereo-less) and stereochemistry-preserved totals.

## When to use

Use when a database release report (e.g. COCONUT October 2020) states aggregate counts (unique flat NPs, stereo molecules) and per-source tables, and those numbers must be independently recomputed and reconciled.

## When NOT to use

- The database already publishes machine-readable per-source and per-release count dumps that you can load directly, making manual reconciliation redundant.
- You need per-molecule provenance or structural deduplication logic rather than only aggregate release-level count verification.

## Inputs

- Per-source entry count table (e.g. Table 1 of a database paper listing counts for each of the 53 public datasets plus manual literature selections)
- Reported release aggregate counts (e.g. October 2020 COCONUT release: 406,076 unique flat NPs; 730,441 NPs with stereochemistry preserved)

## Outputs

- coconut_source_counts.csv — per-source entry counts table with a computed total column
- Verification summary flagging agreement or discrepancy between computed aggregates and reported unique flat / stereo release totals

## How to apply

Assemble the per-source entry counts listed in the release's source table (e.g. COCONUT's 53 public datasets such as AfroDB 874, AfroMalariaDB 252, AfroCancer 365, BitterDB 625, BIOFACQUIM 400, Carotenoids Database 986, AnalytiCon Discovery NPs 4908, ChEBI NPs 14603, plus manually collected literature selections) into a single table using pandas. Compute the column total of aggregated entries, then compare this sum to the reported release totals — for COCONUT October 2020, 406,076 unique flat NPs (unified by stereo-less InChIKey) and 730,441 NPs where stereochemistry was preserved. The per-source sum will exceed the unique flat count because cross-source duplicates are merged during unification; document that excess as evidence of deduplication. Write the per-source counts to CSV and emit a verification summary flagging any discrepancy between computed aggregates and the reported flat/stereo totals, so the release's stated release statistics can be independently confirmed.

## Related tools

- **pandas** (Used to load the per-source entry counts from Table 1, assemble them into a single per-source counts table, and compute the column total.)
- **MongoDB** (Serves as the target store where the assembled per-source counts table and verification summary can be written for cross-checking against the database's unified entries.)

## Examples

```
import pandas as pd; df = pd.read_csv('coconut_source_counts.csv'); print(df['entry_count'].sum())
```

## Evaluation signals

- Computed per-source column total is reported alongside the aggregate sum.
- Verification summary flags discrepancy between summed per-source counts and the reported unique flat / stereo release totals.
- Excess of summed per-source entries over unique flat NPs is explained by cross-source duplicate merging, not by data-entry error.
- Output CSV matches the schema of the source table (one row per dataset, count column, total row).
- Reported release numbers (e.g. 406,076 unique flat NPs) appear unchanged and correctly compared to the computed total.

## Limitations

- Per-source counts are pre-deduplication; reconciliation cannot recover which specific molecules were merged across sources, only that merging occurred.
- Any discrepancy beyond simple duplicate-merging excess must be flagged as unresolved rather than silently accepted.
- Manually collected literature selections in the source table may lack machine-readable provenance, limiting traceability of that portion of the total.

## Evidence

- [methods] October 2020 release totals of unique flat and stereo-preserving NPs: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo (lacking stereoisomers) and 555,897 stereo molecules"
- [methods] Duplicate merging on unification across sources: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry, and the remainder is now included in the parent entry."
- [intro] Unification based on stereo-less InChIKey: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [intro] MongoDB storage of release data and derived information: "All original data, unified NPs and the derived and calculated information are stored in MongoDB."
- [results] Per-source data from 63 collections (September 2024 version): "COCONUT has NP data from 63 sources (as of sept 2024) ranging from supplimentray resources to well curated databases."
