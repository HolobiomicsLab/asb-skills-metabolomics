---
name: cross-source-duplicate-reconciliation
description: Use when aggregating molecule entries from multiple public datasets (e.g.
  AfroDB, ChEBI NPs, BitterDB, AnalytiCon Discovery NPs) and you must detect cross-source
  duplicates, merge them without losing annotations, and reconcile per-source counts
  against release-level unique totals.
when_to_use_negative:
- The dataset contains structures from only a single source with no expected overlap
  — use a simple format/valence QC pipeline instead of cross-source deduplication.
- You need stereochemistry-aware deduplication that keeps stereoisomers as separate
  unique entries; this skill deliberately collapses records on the stereo-less InChIKey,
  so it is wrong if the target schema treats stereoisomers as distinct NPs.
- Structures fail the basic QC thresholds (e.g. >210 heavy atoms, multiple disconnected
  components, pseudo-atoms) before any InChI can be computed reliably — deduplication
  must wait until the parsing/QC stage passes.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: COCONUT compiler (coconut-0.0.1-SNAPSHOT.jar)
  role: Reads molecules from SMILES/SDF/MOL/CSV/TSV, checks errors and connectivity,
    computes molecular properties, unifies NPs across sources by stereo-less InChIKey,
    and stores results in a Mongo database.
  repo: https://github.com/mSorok/COCONUT
- name: CDK
  role: Used to process chemical information and formats, including the InChI/InChIKey
    generation that underpins stereo-less duplicate detection.
  repo: https://github.com/mSorok/COCONUT
- name: MongoDB
  role: Stores all original data, unified NPs and derived/calculated information;
    the uniqueNaturalProduct collection holds merged entries after cross-source reconciliation.
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: ChEMBL structure curation pipeline
  role: Upstream filter that removes 113 structures that cannot be parsed before duplicate
    reconciliation is attempted.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:bea8fb636f53ac8bec007e0c8f8fafd1f4b2cc828bc765bf6677db5fdd0c94a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/COCONUT
  tools:
  - CDK
  - COCONUT compiler (coconut-0.0.1-SNAPSHOT.jar)
  - ChEMBL structure curation pipeline
  - MongoDB
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-source-duplicate-reconciliation

## Summary

Reconcile chemical entries aggregated from many public natural-product sources by unifying duplicates on stereo-less InChIKey identity, merging records into a highly annotated parent entry, and verifying the summed per-source counts against the resulting unique flat and stereo-preserving totals (e.g. 406,076 flat and 730,441 stereo NPs in the October 2020 COCONUT release).

## When to use

Use when aggregating molecule entries from multiple public datasets (e.g. AfroDB, ChEBI NPs, BitterDB, AnalytiCon Discovery NPs) and you must detect cross-source duplicates, merge them without losing annotations, and reconcile per-source counts against release-level unique totals.

## When NOT to use

- The dataset contains structures from only a single source with no expected overlap — use a simple format/valence QC pipeline instead of cross-source deduplication.
- You need stereochemistry-aware deduplication that keeps stereoisomers as separate unique entries; this skill deliberately collapses records on the stereo-less InChIKey, so it is wrong if the target schema treats stereoisomers as distinct NPs.
- Structures fail the basic QC thresholds (e.g. >210 heavy atoms, multiple disconnected components, pseudo-atoms) before any InChI can be computed reliably — deduplication must wait until the parsing/QC stage passes.

## Inputs

- Molecule structure files from each contributing source (SMILES, SDF, MOL, CSV/TSV as read by the COCONUT compiler)
- Per-source entry-count table (Table 1 of the COCONUT paper, e.g. AfroDB 874, ChEBI NPs 14,603)
- Collection of identifier mappings / coconut_ids CSV (e.g. coconut_ids_june2020.csv) for CNPid import
- Reported release-level statistics to reconcile against (406,076 unique flat NPs; 730,441 stereo-preserving NPs)

## Outputs

- coconut_source_counts.csv — per-source entry counts with column totals
- Unified uniqueNaturalProduct collection in MongoDB, one record per stereo-less InChIKey group with the parent (most annotated) entry
- Verification summary flagging discrepancies between summed per-source counts and reported flat/stereo totals
- Set of merged duplicate entries recorded within parent entries, with provenance from all contributing sources

## How to apply

Assemble a per-source counts table (e.g. AfroDB 874, AfroMalariaDB 252, AfroCancer 365, BitterDB 625, BIOFACQUIM 400, Carotenoids Database 986, AnalytiCon Discovery NPs 4,908, ChEBI NPs 14,603) from the COCONUT assembly Table 1, then standardise every structure through the ChEMBL-style quality-control pipeline (heavy-atom count 5–210, keep only the biggest connected structure, correct implicit/explicit hydrogens, Kekulé representation). Compute both original and stereo-less (simple) InChI and InChIKey with CDK, and group entries across sources by identical stereo-less InChIKey so stereoisomers of the same flat skeleton collapse onto one unique record. Merge each duplicate group into one entry, promoting the most highly annotated record to parent and folding the remaining duplicates into it (per the article's 'highly annotated entry has been made the parent entry' rule). Finally, compare the number of unique records against the reported release statistics — 406,076 unique flat NPs and 730,441 NPs preserving stereochemistry — and flag any discrepancy: the excess of summed per-source entries over unique totals quantifies cross-source duplication.

## Related tools

- **COCONUT compiler (coconut-0.0.1-SNAPSHOT.jar)** (Reads molecules from SMILES/SDF/MOL/CSV/TSV, checks errors and connectivity, computes molecular properties, unifies NPs across sources by stereo-less InChIKey, and stores results in a Mongo database.) — https://github.com/mSorok/COCONUT
- **CDK** (Used to process chemical information and formats, including the InChI/InChIKey generation that underpins stereo-less duplicate detection.) — https://github.com/mSorok/COCONUT
- **MongoDB** (Stores all original data, unified NPs and derived/calculated information; the uniqueNaturalProduct collection holds merged entries after cross-source reconciliation.) — https://github.com/mSorok/NaturalProductsOnline
- **ChEMBL structure curation pipeline** (Upstream filter that removes 113 structures that cannot be parsed before duplicate reconciliation is attempted.)

## Examples

```
mongorestore --db=COCONUT --noIndexRestore . ; java -Xmx16288m -jar coconut-0.0.1-SNAPSHOT.jar data sm/sm.tsv fragments/fragment_without_sugar.txt fragments/fragment_with_sugar.txt importCOCONUTids coconut_ids_june2020.csv > logs.txt &
```

## Evaluation signals

- After reconciliation, the number of unique records whose stereo-less InChIKey matches the reported flat-NP total (406,076 in the October 2020 release) is within an accepted tolerance.
- Count of records retaining stereochemistry equals the reported stereo-preserving total (730,441) for the same release.
- For each merged entry, the parent record's annotation level is the highest among duplicates in its InChIKey group (spot-check by annotationLevel index in MongoDB).
- Summed per-source counts from coconut_source_counts.csv exceed the unique totals, and the excess can be explained by merged duplicates — flag any case where summed counts are lower than unique counts.
- No record survives with a structure that fails ChEMBL curation parsing or contains polyfluorinated compounds (64 removed) — these must be excluded before deduplication.

## Limitations

- Structures that cannot be parsed by the ChEMBL structure curation pipeline (113 in total) were removed, so reconciliation cannot operate on those entries.
- Polyfluorinated compounds (64 in total) were removed prior to merging, so they are absent from the reconciled totals.
- 24,880 SuperNatural II molecules failing the additional NP-likeness/NPclassifier/sugar-moiety QC were removed, narrowing which sources contribute duplicates.
- Any molecules that failed to get parsed by RDKit were eliminated from further processing, and molecules marked with a ChEMBL error code of 6 or higher are rejected — both reduce the reconcilable pool.
- The reconciliation only collapses entries on the stereo-less InChIKey, so records differing only by charge/tautomer state or by salt form may not be merged unless they share the same simple InChIKey.

## Evidence

- [intro] unify-on-inchikey: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [methods] merge-parent-rule: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry, and the remainder is now included in the parent entry."
- [intro] qc-thresholds: "its structure is checked for size (between 5 and 210 heavy atoms), connectivity (only the biggest connected structure is kept), presence of pseudo-atoms, if implicit and explicit hydrogens are correct"
- [readme] readme-compiler: [evidence span withheld — anchor status: not_found]
- [readme] readme-indexes: [evidence span withheld — anchor status: external_doc]
