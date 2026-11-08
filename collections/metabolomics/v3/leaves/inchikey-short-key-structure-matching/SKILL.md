---
name: inchikey-short-key-structure-matching
description: Use when building a benchmarking or dereplication set that must join
  a curated structure table (e.g. DNP-derived) to an MS/MS spectral library (e.g.
  GNPS-derived) at the level of unique parent structures rather than individual stereoisomers.
when_to_use_negative:
- The analysis needs to distinguish stereoisomers or stereoisomeric mixtures — the
  short-key join deliberately collapses them
- Structures lack convertible InChI/SMILES or have unresolved charge/protonation states,
  so the first InChIKey layer is not stable across tables
- Input is already a structure-level merged table where each unique short InChIKey
  appears once — re-deduplication adds nothing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: RDKit
  role: Converts chemical structures to their InChIKey (27-character hash of the InChI)
    from which the short 14-character key is extracted
- name: ISDB-DNP
  role: Provides the DNP-backed structural entries whose InChIKeys are harmonized
    against the GNPS spectral library during benchmarking set construction
  repo: github.com/taxonomicallyinformedannotation/tima
- name: GNPS spectral libraries
  role: 'Spectral side of the join: library entries are harmonized by short InChIKey
    and filtered for [M+H]+ adducts before matching'
  repo: https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:6104964329a7a4cdcd93fbd8f60be92736d766d8d202f7d58148a25cc480a202
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS spectral libraries
  - ISDB-DNP
  - RDKit
  - gnfinder
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# inchikey-short-key-structure-matching

## Summary

Match chemical structures between a structural/biological-source database and a spectral library by joining on the first 14 characters (first block) of the InChIKey, so stereoisomers collapse to a single key and duplicate structures are removed during benchmarking set construction.

## When to use

Use when building a benchmarking or dereplication set that must join a curated structure table (e.g. DNP-derived) to an MS/MS spectral library (e.g. GNPS-derived) at the level of unique parent structures rather than individual stereoisomers.

## When NOT to use

- The analysis needs to distinguish stereoisomers or stereoisomeric mixtures — the short-key join deliberately collapses them
- Structures lack convertible InChI/SMILES or have unresolved charge/protonation states, so the first InChIKey layer is not stable across tables
- Input is already a structure-level merged table where each unique short InChIKey appears once — re-deduplication adds nothing

## Inputs

- Curated structural database entries with biological-source metadata (DNP-derived table)
- Filtered MS/MS spectral library entries (GNPS-derived, .mgf with structure annotations)
- Full InChIKey strings (27-character) or InChI/SMILES structures convertible via RDKit

## Outputs

- Short (14-character) InChIKey per entry used as join key
- Merged structure–spectrum–source table deduplicated by short InChIKey
- Benchmarking dataset of 2,107 entries with unique structures and unique biological sources

## How to apply

Convert each candidate structure to its InChIKey, a 27-character hashed version of the full InChI, using RDKit. Extract the first 14 characters (the first layer) as the short InChIKey, which encodes the molecular skeleton independent of stereochemistry and protonation. Harmonize both the structural-source table and the spectral table by this short key, then join them on the short key so structures match across the structure and spectral sides. Deduplicate by keeping only distinct short InChIKeys, which removes duplicate structures and stereoisomers per the paper's convention. The rationale is that GNPS libraries and the DNP differ in stereochemical completeness, so stereo-insensitive keys are the only join that yields a fair one-to-one structure mapping for downstream taxonomic scoring.

## Related tools

- **RDKit** (Converts chemical structures to their InChIKey (27-character hash of the InChI) from which the short 14-character key is extracted)
- **ISDB-DNP** (Provides the DNP-backed structural entries whose InChIKeys are harmonized against the GNPS spectral library during benchmarking set construction) — github.com/taxonomicallyinformedannotation/tima
- **GNPS spectral libraries** (Spectral side of the join: library entries are harmonized by short InChIKey and filtered for [M+H]+ adducts before matching) — https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp

## Examples

```
Rscript tima match_library_structures --input library_all_isdb.mgf --organism taxo.tsv --output matched.mgf --key short_inchikey
```

## Evaluation signals

- The merged table contains exactly 2,107 distinct entries, matching the reported benchmarking dataset size
- Every retained structure has a 14-character short InChIKey and no two rows share the same short key after deduplication
- Spectral entries retained carry parent masses between 100 and 1,500 Da and at least 6 fragments (500 most intense peaks kept for larger spectra)
- All biological sources in the merged table are resolved against the Catalogue of Life and complete down to species level
- Spectral side exports cleanly to .mgf and metadata to .tsv without orphan keys from either side

## Limitations

- The short InChIKey collapses stereoisomers by design, so the benchmarking dataset contains 2,107 entries with no stereoisomer distinction
- 197 benchmarking entries had mass differences > 0.01 Da between experimental parent ion mass and calculated exact mass, indicating structural or annotation errors that short-key matching cannot catch
- The deduplication convention is specific to this study; other pipelines that preserve stereochemistry will yield different set sizes

## Evidence

- [methods] Duplicate structures and stereoisomers removed by distinct short InChIKey: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [methods] Structures converted to InChIKey via RDKit: "converting structures to their InChIKey, a 27-character hashed version of the full InChI"
- [results] Benchmarking dataset size of 2,107 entries: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [other] Task-level validation of 2,107 entries: [evidence span withheld — anchor status: not_found]
- [discussion] Mass-error caveat on benchmarking entries: "a number of entries (197) of the benchmarking dataset were found to have large mass difference (> 0.01 Da)"
