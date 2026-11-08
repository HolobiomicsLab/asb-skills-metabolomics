---
name: per-dataset-entry-tabulation
description: Use when reconstructing the provenance composition of COCONUT (or a similarly
  aggregated NP database) from published per-source counts and validating aggregated
  totals against reported flat/stereo release numbers.
when_to_use_negative:
- The goal is to count entries in the COCONUT 2024 PostgreSQL-derived release (63
  sources, 695,133 molecules) rather than the October 2020 release — different totals
  and source composition apply
- Per-source counts are already provided as a cleaned table and no reconciliation
  against release totals is needed
- Analysis targets stereo-specific or annotation-level subsets (e.g., the 5-star annotation
  system) rather than whole-source entry tabulation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: MongoDB
  role: Store for original per-source data, unified NPs and derived information; sourceNaturalProduct
    and uniqueNaturalProduct collections can be queried per source to cross-check
    tabulated counts
  repo: https://github.com/mSorok/COCONUT
- name: COCONUT compiler (coconut-0.0.1-SNAPSHOT.jar)
  role: Reads molecules from SMILES, SDF, MOL, csv, tsv files per source, curates
    them, and stores everything in a Mongo database from which per-source entry counts
    derive
  repo: https://github.com/mSorok/COCONUT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:5dacac16a3d4a6afc770c92ffe937521be18638da726dbf350cf764091238b14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/COCONUT
  tools:
  - COCONUT compiler (coconut-0.0.1-SNAPSHOT.jar)
  - MongoDB
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# per-dataset-entry-tabulation

## Summary

Aggregates per-source entry counts from the COCONUT Table 1 compilation (53 public datasets plus manual literature selections) and reconciles the summed totals against the October 2020 release statistics of 406,076 unique flat NPs and 730,441 stereochemistry-preserved NPs. It exposes how cross-source duplicate merging explains the excess of summed per-source entries over unique totals.

## When to use

Use when reconstructing the provenance composition of COCONUT (or a similarly aggregated NP database) from published per-source counts and validating aggregated totals against reported flat/stereo release numbers.

## When NOT to use

- The goal is to count entries in the COCONUT 2024 PostgreSQL-derived release (63 sources, 695,133 molecules) rather than the October 2020 release — different totals and source composition apply
- Per-source counts are already provided as a cleaned table and no reconciliation against release totals is needed
- Analysis targets stereo-specific or annotation-level subsets (e.g., the 5-star annotation system) rather than whole-source entry tabulation

## Inputs

- Per-source entry counts from Table 1 of the COCONUT paper (CSV/TSV of dataset name and molecule count, e.g., AfroDB 874, ChEBI NPs 14,603)
- Reported October 2020 COCONUT release aggregate statistics (406,076 unique flat NPs; 730,441 stereo-preserved NPs)
- Optionally, the COCONUT MongoDB dump (e.g., COCONUT2020-07) restored via mongorestore for direct per-source counting

## Outputs

- coconut_source_counts.csv — one row per source dataset with entry counts and a computed total
- Verification summary flagging discrepancies between computed aggregates and the reported flat/stereo release totals

## How to apply

Load the per-source entry counts from Table 1 of the COCONUT paper (e.g., AfroDB 874, AfroMalariaDB 252, AfroCancer 365, BitterDB 625, BIOFACQUIM 400, Carotenoids Database 986, AnalytiCon Discovery NPs 4,908, ChEBI NPs 14,603) with pandas, assembling the 53 public dataset rows plus the manually collected literature selections into a single counts table with a computed column total. Then reconcile the summed entries against the October 2020 release statistics: 406,076 unique flat NPs (unified by stereo-less InChIKey) and 730,441 NPs where stereochemistry was preserved. Any excess of summed per-source counts over unique totals is expected because duplicates from different provenance are merged into one parent entry — the highly annotated entry becomes the parent. Write the per-source table to coconut_source_counts.csv and emit a verification summary flagging discrepancies against the reported totals.

## Related tools

- **MongoDB** (Store for original per-source data, unified NPs and derived information; sourceNaturalProduct and uniqueNaturalProduct collections can be queried per source to cross-check tabulated counts) — https://github.com/mSorok/COCONUT
- **COCONUT compiler (coconut-0.0.1-SNAPSHOT.jar)** (Reads molecules from SMILES, SDF, MOL, csv, tsv files per source, curates them, and stores everything in a Mongo database from which per-source entry counts derive) — https://github.com/mSorok/COCONUT

## Examples

```
import pandas as pd; df = pd.read_csv('coconut_table1.csv'); total = df['entries'].sum(); print(total, 'vs 406076 flat / 730441 stereo')
```

## Evaluation signals

- Computed column total of per-source entries must exceed the 406,076 unique flat NP count, with the excess attributable to cross-source duplicate merging
- The 730,441 stereo-preserved total should be reported alongside the 406,076 flat total; the flat total arises from unifying NPs by stereo-less InChIKey identity
- coconut_source_counts.csv must contain the 53 public dataset rows plus the manual literature selections and parse cleanly with pandas
- Verification summary should emit zero unexplained discrepancies (all differences explained by duplicate merging)
- If cross-checked against the MongoDB dump, per-source counts from the sourceNaturalProduct collection should match the Table 1 values

## Limitations

- Counts pertain to the October 2020 release; later releases differ (e.g., 63 sources and 695,133 molecules as of September 2024, with 89,593 non-stereo and 555,897 stereo molecules)
- Curation removed specific entries (64 polyfluorinated compounds, 113 structures unparsable by the ChEMBL structure curation pipeline, and 24,880 SuperNatural II molecules failing additional QC), so raw source counts will not equal release totals
- The excess of summed per-source entries over unique totals depends on the duplicate-merge policy (highly annotated entry made parent), so totals are not simply reproducible by summation

## Evidence

- [methods] Duplicates merged explain totals: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry, and the remainder is now included in the parent entry."
- [intro] Unification by stereo-less InChIKey: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [intro] MongoDB storage of sources: "All original data, unified NPs and the derived and calculated information are stored in MongoDB."
- [readme] README: loading the MongoDB dump: [evidence span withheld — anchor status: not_found]
