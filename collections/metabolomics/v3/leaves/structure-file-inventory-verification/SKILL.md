---
name: structure-file-inventory-verification
description: Use when a data record asserts a specific number and provenance split
  of structure files (e.g. .pdb files from PDB vs AlphaFold DB) in a deposit and the
  deposit contents must be enumerated and reconciled against that statement.
when_to_use_negative:
- The archive contains non-.pdb structure formats (e.g. .cif, .pdbqt) that the Data
  Records statement does not cover — counting them against the 408-file claim would
  produce spurious mismatches.
- The goal is to validate structural quality or model confidence of the peptide hormone
  structures rather than to reconcile the file inventory against the deposit's stated
  counts.
- Structure files are accessed individually from PDB or AlphaFold DB directly instead
  of the consolidated HORDB 'Peptide structure.zip' package, since provenance split
  claims only apply to that deposit.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: manual expert review
  role: Used to classify each enumerated .pdb file by provenance (PDB-derived vs AlphaFold
    DB-derived) per the deposit's Data Records description
- name: HORDB source code
  role: Repository accompanying the manuscript; datasets referenced by the workflow
    are mirrored in /datasets/ and on the HORDB download page
  repo: CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:561e79e2f961e4dbb161b902f17ab9c96f46ba05025f281ec6d1367e290134da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CPU-HORDB/HORDB
  tools:
  - HORDB source code
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# structure-file-inventory-verification

## Summary

A verification skill for enumerating and classifying .pdb three-dimensional structure files deposited in the HORDB 'Peptide structure.zip' figshare package, checking that per-source counts (PDB-derived vs AlphaFold DB-derived) match the Data Records statement of 408 total files (54 PDB + 354 AlphaFold DB).

## When to use

Use when a data record asserts a specific number and provenance split of structure files (e.g. .pdb files from PDB vs AlphaFold DB) in a deposit and the deposit contents must be enumerated and reconciled against that statement.

## When NOT to use

- The archive contains non-.pdb structure formats (e.g. .cif, .pdbqt) that the Data Records statement does not cover — counting them against the 408-file claim would produce spurious mismatches.
- The goal is to validate structural quality or model confidence of the peptide hormone structures rather than to reconcile the file inventory against the deposit's stated counts.
- Structure files are accessed individually from PDB or AlphaFold DB directly instead of the consolidated HORDB 'Peptide structure.zip' package, since provenance split claims only apply to that deposit.

## Inputs

- figshare deposit archive 'Peptide structure.zip' from https://doi.org/10.6084/m9.figshare.c.5522241
- Deposit Data Records description defining PDB-derived vs AlphaFold DB-derived file provenance

## Outputs

- structure_file_counts.csv — table with columns: source category, file count
- Validation report of matched/mismatched counts vs the 54 PDB + 354 AlphaFold DB = 408 total statement

## How to apply

Download the 'Peptide structure.zip' package from the HORDB figshare deposit (https://doi.org/10.6084/m9.figshare.c.5522241) and extract it locally. Enumerate all .pdb three-dimensional structure files it contains, then classify each file by provenance — PDB-derived vs AlphaFold DB-derived — according to the deposit's Data Records description (e.g. via naming conventions or metadata distinguishing the two sources). Tabulate counts per source category in a two-column table (source category, file count) and write it to structure_file_counts.csv. Validation hinges on the Data Records claim: the counts must equal 54 PDB-derived and 354 AlphaFold DB-derived .pdb files (408 total); any mismatch must be explicitly reported in the output table rather than silently accepted. Classification is done by manual expert review, so ambiguous files should be flagged rather than force-assigned.

## Related tools

- **manual expert review** (Used to classify each enumerated .pdb file by provenance (PDB-derived vs AlphaFold DB-derived) per the deposit's Data Records description)
- **HORDB source code** (Repository accompanying the manuscript; datasets referenced by the workflow are mirrored in /datasets/ and on the HORDB download page) — CPU-HORDB/HORDB

## Examples

```
unzip Peptide_structure.zip -d peptide_structures && find peptide_structures -name '*.pdb' | wc -l  # expect 408
```

## Evaluation signals

- Total enumerated .pdb file count equals 408, matching the Data Records statement.
- PDB-derived subset count equals exactly 54 and AlphaFold DB-derived subset count equals exactly 354.
- Every file in the archive is assigned to exactly one provenance category (no unclassified or double-counted files).
- structure_file_counts.csv has the required schema: columns for source category and file count.
- Any count mismatch vs the Data Records claim is reported in the output table rather than silently dropped.

## Limitations

- Validation thresholds are tied to a single frozen deposit state; if the figshare package is updated, the 54/354/408 expectation may no longer hold and must be re-checked against the current Data Records text.
- Provenance classification relies on manual expert review against the Data Records description, so files with ambiguous naming or missing provenance cues may be hard to assign.
- The skill verifies inventory counts only — it does not validate the biological correctness or structural quality of the individual peptide hormone structures.

## Evidence

- [other] Data Records statement of 408 files: [evidence span withheld — anchor status: not_found]
- [abstract] figshare deposit location: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] datasets directory in repository: "/datasets/ Contains all datasets from HORDB"
