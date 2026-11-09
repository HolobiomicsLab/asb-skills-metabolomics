---
name: natural-products-source-count-aggregation
description: Use when you need to reproduce or audit COCONUT's dataset statistics
  for a specific release — i.e., reconstruct Table 1 per-source counts and verify
  them against reported totals of unique flat and stereochemistry-preserving NPs.
when_to_use_negative:
- The target release is not the October 2020 COCONUT release — later releases have
  different provenance (e.g., 63 sources as of September 2024) and different totals
  (695,133 molecules with 89,593 non-stereo and 555,897 stereo), so Oct-2020 reference
  totals will not reconcile
- The goal is per-molecule provenance tracing (which source/organism a specific NP
  came from) rather than aggregate per-source counts
- The source datasets still need structure curation or quality control (e.g., removal
  of the 24,880 SuperNatural II molecules, 64 polyfluorinated compounds, or 113 unparsable
  structures) — this skill assumes post-curation counts
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MongoDB
  role: Storage and query layer for COCONUT's original data, unified NPs, and derived
    information; used to cross-check per-source and unique counts after loading a
    release dump
- name: pandas
  role: Builds the per-source counts table from Table 1 values and computes the aggregated-entry
    column total
- name: COCONUT compiler JAR
  role: Compiles molecules from SMILES/SDF/MOL/csv/tsv files into the Mongo database
    from which the source and unique NP counts are derived
  repo: https://zenodo.org/record/3695455
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:0a7e5f2471cd500d5b78975811144af983729fa9ea4154141bc5d7f3c0a235cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COCONUT compiler JAR
  - MongoDB
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# natural-products-source-count-aggregation

## Summary

Reconstructs COCONUT's provenance accounting by assembling the per-source entry counts of its public datasets (e.g., AfroDB 874, BitterDB 625, ChEBI NPs 14,603) plus manual literature selections into a single table, and reconciles the summed totals against the release-level unique counts obtained by merging duplicates via stereo-less InChIKey.

## When to use

Use when you need to reproduce or audit COCONUT's dataset statistics for a specific release — i.e., reconstruct Table 1 per-source counts and verify them against reported totals of unique flat and stereochemistry-preserving NPs.

## When NOT to use

- The target release is not the October 2020 COCONUT release — later releases have different provenance (e.g., 63 sources as of September 2024) and different totals (695,133 molecules with 89,593 non-stereo and 555,897 stereo), so Oct-2020 reference totals will not reconcile
- The goal is per-molecule provenance tracing (which source/organism a specific NP came from) rather than aggregate per-source counts
- The source datasets still need structure curation or quality control (e.g., removal of the 24,880 SuperNatural II molecules, 64 polyfluorinated compounds, or 113 unparsable structures) — this skill assumes post-curation counts

## Inputs

- Per-source entry counts from Table 1 of the COCONUT paper (53 public datasets, e.g., AfroDB 874, ChEBI NPs 14,603)
- Manually collected literature selections count (not tied to a public dataset)
- Reported release statistics for the October 2020 COCONUT release: 406,076 unique flat NPs and 730,441 NPs with stereochemistry
- Optional: MongoDB dump of a COCONUT release (e.g., COCONUT2020-07 from Zenodo) with sourceNaturalProduct and uniqueNaturalProduct collections

## Outputs

- coconut_source_counts.csv — per-source entry counts with a column total of aggregated entries
- Verification summary flagging discrepancies between computed aggregates and the reported 406,076 flat / 730,441 stereo totals

## How to apply

Load the per-source entry counts from Table 1 of the COCONUT paper (AfroDB 874, AfroMalariaDB 252, AfroCancer 365, BitterDB 625, BIOFACQUIM 400, Carotenoids Database 986, AnalytiCon Discovery NPs 4,908, ChEBI NPs 14,603, etc.) with pandas, plus a row for manually collected literature selections. Compute the column total of aggregated entries, then reconcile it against the reported October 2020 release statistics (406,076 unique flat NPs unified by stereo-less InChIKey; 730,441 NPs with stereochemistry preserved). The summed per-source counts legitimately exceed the unique totals because NPs from different provenance are unified when their InChI keys without stereochemistry are identical, and duplicates are merged with the most highly annotated entry as the parent — so explain the excess as cross-source duplicate merging. If you reload the MongoDB dump (mongorestore of the COCONUT Zenodo dump, e.g., COCONUT2020-07), cross-check counts by querying the uniqueNaturalProduct and sourceNaturalProduct collections after recreating the source/inchikey indexes described in the README. Write the per-source table to coconut_source_counts.csv and emit a verification summary flagging any discrepancy between computed aggregates and reported flat/stereo totals.

## Related tools

- **MongoDB** (Storage and query layer for COCONUT's original data, unified NPs, and derived information; used to cross-check per-source and unique counts after loading a release dump)
- **pandas** (Builds the per-source counts table from Table 1 values and computes the aggregated-entry column total)
- **COCONUT compiler JAR** (Compiles molecules from SMILES/SDF/MOL/csv/tsv files into the Mongo database from which the source and unique NP counts are derived) — https://zenodo.org/record/3695455

## Examples

```
mongo --port 27017 COCONUT2020-07 --quiet --eval 'db.uniqueNaturalProduct.countDocuments({}); db.sourceNaturalProduct.aggregate([{$group:{_id:"$source",n:{$sum:1}}}])'
```

## Evaluation signals

- The summed per-source entry counts exceed the unique release totals, and the excess is attributable to cross-source duplicate merging on stereo-less InChIKey
- The unique flat NP total (406,076) is strictly less than the stereochemistry-preserved total (730,441) for the October 2020 release
- The counts table contains one row per public dataset in Table 1 (53 datasets) plus a manual literature selections entry
- A verification summary is emitted and reports zero unresolved discrepancies against the reported flat/stereo totals
- When cross-checking in MongoDB, uniqueNaturalProduct counts match the flat NP total and sourceNaturalProduct counts match the summed per-source entries

## Limitations

- Counts are release-specific: the article itself reports 695,133 total molecules (89,593 non-stereo, 555,897 stereo) with 63 sources for the September 2024 release, so reconciling against October 2020 numbers only holds for that snapshot
- Summed per-source counts can never equal unique counts because duplicates are merged and the highly annotated entry becomes the parent
- Curation removals (24,880 SuperNatural II molecules failing NP-likeness/NPclassifier/sugar-moiety checks, 64 polyfluorinated compounds, 113 structures unparsable by the ChEMBL structure curation pipeline) mean raw source downloads will not match Table 1 counts
- MongoDB dumps should be restored with --noIndexRestore and indexes recreated manually, as documented in the README

## Evidence

- [intro] unification-on-inchikey: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [methods] duplicate-merge-policy: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry"
- [abstract] aggregated-open-sources: "an aggregated dataset of elucidated and predicted NPs collected from open sources"
- [readme] mongorestore-dump: [evidence span withheld — anchor status: not_found]
- [readme] compiler-inputs: [evidence span withheld — anchor status: not_found]
