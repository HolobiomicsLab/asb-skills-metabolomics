---
name: chemical-database-curation-audit
description: Use when reproducing or verifying aggregate counts of a curated chemical
  database assembled from multiple public sources, or when reconciling per-source
  entry totals against published unique-molecule statistics after deduplication.
when_to_use_negative:
- Input is already a deduplicated unique-molecule table rather than per-source raw
  counts — the reconciliation logic would not apply
- The target database lacks published aggregate statistics (flat vs. stereo totals)
  to reconcile against, so no audit target exists
- The analysis goal is structural annotation or property computation (descriptors,
  fingerprints, classification) rather than count-level database assembly verification
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_2258
tools:
- name: MongoDB
  role: Stores all original data, unified NPs and derived information; a dump (COCONUT2020-07)
    can be restored and queried to recount sourceNaturalProduct and uniqueNaturalProduct
    entries directly
  repo: github.com/mSorok/COCONUT
- name: CDK
  role: Library used to process chemical information and formats during the structure
    quality-control that shapes the aggregate counts
  repo: github.com/mSorok/COCONUT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:ab8c3e20719683df0b695b1a82c374672443a8e6c5fa25f57d041d7324b54fce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CDK
  - MongoDB
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# chemical-database-curation-audit

## Summary

Audits the assembly of a large aggregated chemical database (here, COCONUT) by rebuilding per-source entry counts, reproducing the aggregation and deduplication logic (stereo-less InChIKey unification), and reconciling computed totals against reported release statistics.

## When to use

Use when reproducing or verifying aggregate counts of a curated chemical database assembled from multiple public sources, or when reconciling per-source entry totals against published unique-molecule statistics after deduplication.

## When NOT to use

- Input is already a deduplicated unique-molecule table rather than per-source raw counts — the reconciliation logic would not apply
- The target database lacks published aggregate statistics (flat vs. stereo totals) to reconcile against, so no audit target exists
- The analysis goal is structural annotation or property computation (descriptors, fingerprints, classification) rather than count-level database assembly verification

## Inputs

- Per-source entry counts from the database paper's aggregation table (Table 1, 53 public datasets plus manual literature selections)
- Published release aggregate statistics (October 2020 COCONUT: 406,076 unique flat NPs; 730,441 stereochemistry-preserved NPs)
- Optional: MongoDB dump of the source database (e.g., COCONUT2020-07) for direct counting

## Outputs

- coconut_source_counts.csv — per-source counts table with a column total of aggregated entries
- Verification summary flagging discrepancies between computed aggregates and reported flat/stereo release totals
- Reconciliation note attributing excess per-source counts to cross-source duplicate merging

## How to apply

Load the per-source entry counts from the database paper's aggregation table (e.g., COCONUT Table 1: AfroDB 874, AfroMalariaDB 252, AfroCancer 365, BitterDB 625, BIOFACQUIM 400, Carotenoids Database 986, AnalytiCon Discovery NPs 4,908, ChEBI NPs 14,603) with pandas and add the manually collected literature selections. Compute a column total of aggregated entries and compare it to the reported release statistics — for the October 2020 COCONUT release, 406,076 unique flat NPs (unified by stereo-less InChIKey) and 730,441 NPs with stereochemistry preserved. Explain any excess of summed per-source counts over unique totals by cross-source duplicate merging, where the most highly annotated entry becomes the parent entry. Account for curation-side removals that shrink totals: quality control on structure size (5–210 heavy atoms), connectivity (keeping only the biggest connected structure), pseudo-atoms, hydrogens, bonds and valences; the removal of 64 polyfluorinated compounds; removal of 113 structures unparsable by the ChEMBL structure curation pipeline; and 24,880 SuperNatural II molecules dropped for failing the additional NP-likeness/NPclassifier/sugar-moiety filter. Emit a verification summary flagging any discrepancy between computed aggregates and reported flat/stereo release totals.

## Related tools

- **MongoDB** (Stores all original data, unified NPs and derived information; a dump (COCONUT2020-07) can be restored and queried to recount sourceNaturalProduct and uniqueNaturalProduct entries directly) — github.com/mSorok/COCONUT
- **CDK** (Library used to process chemical information and formats during the structure quality-control that shapes the aggregate counts) — github.com/mSorok/COCONUT

## Examples

```
mongorestore --db=COCONUT --noIndexRestore COCONUT2020-07/COCONUT2020-07/ && mongo --quiet COCONUT --eval 'db.sourceNaturalProduct.count(); db.uniqueNaturalProduct.count()'
```

## Evaluation signals

- Computed column total of per-source entries exceeds the 406,076 unique flat NP total, with the excess attributable to cross-source duplicate merging
- Unique flat count (406,076, stereo-less InChIKey unification) is strictly less than the stereochemistry-preserved total (730,441) for the October 2020 release
- Per-source counts reproduce the published Table 1 values exactly (e.g., AfroDB 874, ChEBI NPs 14,603, AnalytiCon Discovery NPs 4,908)
- Verification summary flags zero unexplained discrepancy between computed aggregates and reported flat/stereo release totals
- When recounting from the MongoDB dump, sourceNaturalProduct entry counts exceed uniqueNaturalProduct counts, consistent with deduplication

## Limitations

- Cross-source duplicate merging makes the summed per-source totals an upper bound; the excess over unique totals depends on merge decisions, so exact reconciliation may require per-entry provenance
- Curation removals (64 polyfluorinated compounds, 113 structures unparsable by the ChEMBL pipeline, 24,880 SuperNatural II molecules, RDKit-unparsable molecules, ChEMBL error code ≥ 6 rejections) reduce totals and must be accounted for when reconciling counts
- A later release snapshot (September 2024) reports 63 sources and 695,133 molecules (89,593 non-stereo, 555,897 stereo), so statistics must be tied to a specific release date

## Evidence

- [methods] Duplicate merging rationale: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry, and the remainder is now included in the parent entry."
- [intro] Structure QC thresholds: "its structure is checked for size (between 5 and 210 heavy atoms), connectivity (only the biggest connected structure is kept), presence of pseudo-atoms, if implicit and explicit hydrogens are correct"
- [intro] Unification key: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [methods] Curation removals: "Polyfluorinated compounds (64 in total) were removed"
- [readme] MongoDB restore procedure: [evidence span withheld — anchor status: not_found]
