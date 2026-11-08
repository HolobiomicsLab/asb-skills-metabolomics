---
name: data-record-count-validation
description: Use when a data descriptor asserts specific file/record counts for a
  deposited package and you need to reproduce, classify, and verify those counts from
  the deposit itself.
when_to_use_negative:
- The deposit does not come with a published count statement to validate against —
  enumeration alone without a reference number is not a validation task
- The target files require content-level curation (e.g. sequence or activity annotation
  of HORDB's 6024 entries) rather than provenance classification and counting
- Files are in formats other than .pdb structure files (e.g. peptide drug or pharmaceutical
  preparation records), where the Data Records provenance split does not apply
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_2815
tools:
- name: manual expert review
  role: Used to classify each .pdb file by provenance (PDB-derived vs AlphaFold DB-derived)
    according to the Data Records description and to verify counts
- name: HORDB figshare deposit
  role: Source of the 'Peptide structure.zip' package containing the 408 .pdb structure
    files to be enumerated
  repo: https://doi.org/10.6084/m9.figshare.c.5522241
- name: CPU-HORDB/HORDB
  role: HORDB source code repository; /datasets/ contains all datasets from HORDB
    also referenced for download
  repo: CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:7e6d2bfd5b174821382575f5951b79a2e5a0c4067e5fa57e2510ab29cc55a380
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CPU-HORDB/HORDB
  - HORDB figshare deposit
  - manual expert review
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_2815
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# data-record-count-validation

## Summary

Enumerate and classify the data records in a deposited archive (here the .pdb 3D structure files in HORDB's 'Peptide structure.zip' from figshare) and validate the tallied counts against the numbers asserted in the publication's Data Records section.

## When to use

Use when a data descriptor asserts specific file/record counts for a deposited package and you need to reproduce, classify, and verify those counts from the deposit itself.

## When NOT to use

- The deposit does not come with a published count statement to validate against — enumeration alone without a reference number is not a validation task
- The target files require content-level curation (e.g. sequence or activity annotation of HORDB's 6024 entries) rather than provenance classification and counting
- Files are in formats other than .pdb structure files (e.g. peptide drug or pharmaceutical preparation records), where the Data Records provenance split does not apply

## Inputs

- Deposited archive 'Peptide structure.zip' from the HORDB figshare collection (DOI 10.6084/m9.figshare.c.5522241)
- Data Records description from the HORDB article stating expected provenance counts (54 PDB-derived, 354 AlphaFold DB-derived .pdb files)

## Outputs

- structure_file_counts.csv: table with columns 'source category' and 'file count' (PDB-derived = 54, AlphaFold DB-derived = 354, total = 408)
- Mismatch report embedded in the output table if tabulated counts do not match the Data Records statement

## How to apply

Download the deposit package (e.g. 'Peptide structure.zip' from the HORDB figshare collection https://doi.org/10.6084/m9.figshare.c.5522241) and extract it locally. Enumerate the target file type — here every .pdb three-dimensional structure file of peptide hormones — with a recursive file listing. Classify each file by provenance according to the Data Records description: structures derived from PDB (54 expected) versus models derived from AlphaFold DB (354 expected). Tabulate counts per source category into a table (source category, file count) and write it to structure_file_counts.csv. Treat the Data Records statement as the ground truth: any mismatch between the tabulated counts and the asserted 54 + 354 = 408 total must be explicitly reported in the output table rather than silently adjusted.

## Related tools

- **manual expert review** (Used to classify each .pdb file by provenance (PDB-derived vs AlphaFold DB-derived) according to the Data Records description and to verify counts)
- **HORDB figshare deposit** (Source of the 'Peptide structure.zip' package containing the 408 .pdb structure files to be enumerated) — https://doi.org/10.6084/m9.figshare.c.5522241
- **CPU-HORDB/HORDB** (HORDB source code repository; /datasets/ contains all datasets from HORDB also referenced for download) — CPU-HORDB/HORDB

## Examples

```
unzip 'Peptide structure.zip' -d peptide_structures && find peptide_structures -name '*.pdb' | wc -l
```

## Evaluation signals

- Total number of .pdb files enumerated from the extracted archive equals 408
- PDB-derived count equals 54 and AlphaFold DB-derived count equals 354
- Sum of per-category counts in structure_file_counts.csv equals the row total, and every file is assigned to exactly one provenance category
- Any deviation from the Data Records figures is explicitly flagged in the output table (no silent reconciliation)
- Every .pdb file in the archive appears in the tabulation — no missing or double-counted files

## Limitations

- Validation is only as reliable as the Data Records statement; provenance classification relies on the deposit's description, so a mislabelled file in the deposit propagates into the tabulation
- Counts pertain to the 'Peptide structure.zip' package only; HORDB's other record types (5729 peptide hormones, 40 peptide drugs, 255 pharmaceutical preparations) are not covered by this check

## Evidence

- [results] The 'Peptide structure.zip' package contains 408 three-dimensional structure files of peptide hormones, comprising 54 structures from PDB and 354 files from AlphaFold DB.: [evidence span withheld — anchor status: not_found]
- [abstract] All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 .: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [abstract] The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information.: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information."
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
