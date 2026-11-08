---
name: cheminformatics-structure-standardization
description: Use when aggregating small-molecule structures from heterogeneous sources
  (SDF, SMILES, csv/tsv) and you need consistent curation rules plus deduplication
  into unique entries keyed on InChI keys.
when_to_use_negative:
- Input is already a curated, deduplicated database dump restored into MongoDB — re-running
  the full compile from scratch would duplicate the unification work; use recomputeMissing
  or index-creation paths instead
- You need stereochemistry-aware matching across the whole catalog — unification keys
  on InChI keys WITHOUT stereochemistry, so stereoisomers collapse into one parent
  entry
- Structures are not natural products (e.g., arbitrary screening libraries) — SuperNatural
  II-style entries must pass the additional NP-likeness/NPclassifier/sugar-moiety
  gate, and the pipeline's QC thresholds (5–210 heavy atoms) target NP-sized molecules
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Parsing molecules, computing standard InChI and stripping stereochemistry
    layers to derive the stereochemistry-free InChI key; unparsable molecules are
    eliminated
- name: CDK
  role: Processing chemical information and formats, computing molecular properties/descriptors
    during curation and standardization
- name: MongoDB
  role: Staging sourceNaturalProduct entries and storing the unified uniqueNaturalProduct
    collection with hashed/simpleInchiKey indexes
  repo: https://github.com/mSorok/NaturalProductsOnline
- name: COCONUT compiler
  role: Standalone JAR that reads molecules from various file types, checks errors
    and connectivity, computes molecular parameters, and stores everything in Mongo
  repo: https://github.com/mSorok/COCONUT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00478-9
    title: 'COCONUT online: Collection of Open Natural Products database'
  source_content_hash: sha256:70cf3a516d16b523272cab8a6a4df62b41e4b8a527f6f0dae079ce58079d9401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mSorok/NaturalProductsOnline
  tools:
  - CDK
  - COCONUT compiler
  - MongoDB
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cheminformatics-structure-standardization

## Summary

COCONUT's structure curation and unification pipeline: quality-control molecules from open NP sources (heavy-atom size, connectivity, pseudo-atoms, valences, Kekulé form), remove unparsable/polyfluorinated entries, then unify duplicates by stereochemistry-free InChI key into unique entries with CNP identifiers while preserving original stereochemical variants.

## When to use

Use when aggregating small-molecule structures from heterogeneous sources (SDF, SMILES, csv/tsv) and you need consistent curation rules plus deduplication into unique entries keyed on InChI keys.

## When NOT to use

- Input is already a curated, deduplicated database dump restored into MongoDB — re-running the full compile from scratch would duplicate the unification work; use recomputeMissing or index-creation paths instead
- You need stereochemistry-aware matching across the whole catalog — unification keys on InChI keys WITHOUT stereochemistry, so stereoisomers collapse into one parent entry
- Structures are not natural products (e.g., arbitrary screening libraries) — SuperNatural II-style entries must pass the additional NP-likeness/NPclassifier/sugar-moiety gate, and the pipeline's QC thresholds (5–210 heavy atoms) target NP-sized molecules

## Inputs

- Per-source molecular structure files (SMILES, SDF, MOL, csv, tsv) from open NP databases (e.g., ChEBI NPs, AfroDB, BIOFACQUIM, AnalytiCon Discovery NPs)
- COCONUT release 2024 PostgreSQL database snapshot as the primary curated source
- Synthetic-molecule reference set (e.g., ZINC15 sm.tsv) for NP-likeness score recalculation

## Outputs

- uniqueNaturalProduct MongoDB collection of unified entries, each with a CNP-prefixed 7-digit identifier, clean_smiles, inchi, inchikey, molecular_formula, fragments, and annotationLevel
- Structured JSON export of the unified UniqueNaturalProduct collection
- Rejection log of molecules failing RDKit parsing or carrying ChEMBL error code ≥ 6

## How to apply

Load per-source structures into a MongoDB staging collection (sourceNaturalProduct), then run the ChEMBL structure curation pipeline and RDKit/CDK parsing; reject molecules failing to parse or with ChEMBL error code ≥ 6, filter to 5–210 heavy atoms, keep only the biggest connected structure, and standardize pseudo-atoms, implicit/explicit hydrogens, bonds, valences and Kekulé representation. Remove polyfluorinated compounds (64 in COCONUT 2024) and, for SuperNatural II entries only, apply the extra NP-likeness/NPclassifier/sugar-moiety filter. Compute the standard InChI with RDKit/CDK, strip stereochemistry layers to obtain the stereochemistry-free InChI key, and group molecules by identical key: merge duplicates into one entry with the most highly annotated entry as parent, nesting the others' original stereochemical structures within it. Assign each unique natural product a CNP-prefixed 7-digit identifier (e.g., CNP0228556) and export the uniqueNaturalProduct collection from MongoDB to structured JSON.

## Related tools

- **RDKit** (Parsing molecules, computing standard InChI and stripping stereochemistry layers to derive the stereochemistry-free InChI key; unparsable molecules are eliminated)
- **CDK** (Processing chemical information and formats, computing molecular properties/descriptors during curation and standardization)
- **MongoDB** (Staging sourceNaturalProduct entries and storing the unified uniqueNaturalProduct collection with hashed/simpleInchiKey indexes) — https://github.com/mSorok/NaturalProductsOnline
- **COCONUT compiler** (Standalone JAR that reads molecules from various file types, checks errors and connectivity, computes molecular parameters, and stores everything in Mongo) — https://github.com/mSorok/COCONUT

## Examples

```
java -Xmx16288m -jar coconut-0.0.1-SNAPSHOT.jar data sm/sm.tsv fragments/fragment_without_sugar.txt fragments/fragment_with_sugar.txt importCOCONUTids coconut_ids_june2020.csv > logs.txt &
```

## Evaluation signals

- No two entries in the uniqueNaturalProduct collection share the same stereochemistry-free InChI key
- Every entry carries a unique CNP-prefixed 7-digit identifier, with stereochemical variants retained under the parent (catalog: 695,133 molecules → 415,768 unique parent structures)
- Every kept structure respects the QC envelope: 5–210 heavy atoms, single biggest connected component, correct implicit/explicit hydrogens, valid valences, Kekulé representation assigned
- Polyfluorinated (64) and ChEMBL-pipeline-unparsable (113) structures are absent from the final collection
- mongorestore + createIndex commands on simpleInchiKey/originalInchiKey succeed, confirming schema fields (simpleInchi, originalSmiles, absoluteSmiles, idInSource) are populated

## Limitations

- Structures that cannot be parsed by the ChEMBL structure curation pipeline (113 in COCONUT 2024) are removed entirely rather than repaired
- Unification deliberately ignores stereochemistry, so stereoisomers of the same parent share one CNP identifier, distinguished only by .0-style variant suffixes
- Molecules failing RDKit parsing are eliminated from further processing; error-code-≥6 entries are only rejected and kept for future reference, not auto-fixed
- The CNPid generation step is flagged by the authors as 'not recommended' for external users; identifiers should be imported from the released CSV instead

## Evidence

- [intro] QC envelope: "its structure is checked for size (between 5 and 210 heavy atoms), connectivity (only the biggest connected structure is kept), presence of pseudo-atoms"
- [intro] InChI-key unification: "Then, NPs from different provenance are unified based on the identity of their InChI keys without stereochemistry."
- [methods] duplicate merging rule: "Duplicates have been merged into one entry and the highly annotated entry has been made the parent entry"
- [methods] unparsable removal: "Structures that cannot be parsed by the ChEMBL structure curation pipeline (113 in total) have been removed."
- [results] RDKit rejection: "Any molecules that failed to get parsed by RDKit were eliminated from further processing."
- [methods] catalog totals: "The database catalogs a total of 695,133 chemical molecules, with 89,593 being non-stereo"
- [readme] README compiler scope: [evidence span withheld — anchor status: not_found]
- [readme] README indexes: " db.sourceNaturalProduct.createIndex( {simpleInchiKey:1})
 db.sourceNaturalProduct.createIndex( {originalInchiKey:1})"
