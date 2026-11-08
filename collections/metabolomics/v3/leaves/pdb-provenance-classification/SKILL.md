---
name: pdb-provenance-classification
description: Use when a deposit contains a mixed collection of .pdb 3D structure files
  and each file must be attributed to its original source (experimental PDB entries
  vs AlphaFold DB predictions) and counted per category.
when_to_use_negative:
- The deposit contains non-.pdb structure formats (e.g. .cif, .mmcif) that are not
  part of the 'Peptide structure.zip' inventory being audited.
- The task requires structural quality assessment or model confidence scoring (e.g.
  pLDDT analysis) rather than simple provenance classification and counting.
- Working with the HORDB entry-level datasets (/datasets/) rather than the 3D structure
  archive; this skill applies specifically to the 'Peptide structure.zip' package.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_2815
tools:
- name: HORDB
  role: Source of the 'Peptide structure.zip' deposit; its Data Records description
    defines the PDB vs AlphaFold DB provenance split (54 vs 354) used as the classification
    ground truth
  repo: CPU-HORDB/HORDB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:e496cc1c88d9e01930285264dd6305e8f9988c0e879157e94523278865dc3263
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CPU-HORDB/HORDB
  tools:
  - HORDB
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_2815
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pdb-provenance-classification

## Summary

Enumerates and classifies the .pdb three-dimensional structure files of peptide hormones distributed in the HORDB 'Peptide structure.zip' figshare deposit, attributing each file to either the Protein Data Bank (PDB) or AlphaFold DB and tabulating the counts (expected: 54 PDB-derived + 354 AlphaFold DB-derived = 408 total).

## When to use

Use when a deposit contains a mixed collection of .pdb 3D structure files and each file must be attributed to its original source (experimental PDB entries vs AlphaFold DB predictions) and counted per category.

## When NOT to use

- The deposit contains non-.pdb structure formats (e.g. .cif, .mmcif) that are not part of the 'Peptide structure.zip' inventory being audited.
- The task requires structural quality assessment or model confidence scoring (e.g. pLDDT analysis) rather than simple provenance classification and counting.
- Working with the HORDB entry-level datasets (/datasets/) rather than the 3D structure archive; this skill applies specifically to the 'Peptide structure.zip' package.

## Inputs

- 'Peptide structure.zip' archive from the HORDB figshare deposit (https://doi.org/10.6084/m9.figshare.c.5522241)
- Deposit Data Records description defining per-file provenance (PDB-derived vs AlphaFold DB-derived)

## Outputs

- structure_file_counts.csv — a table with columns for source category (PDB-derived, AlphaFold DB-derived) and file count
- A complete inventory of the 408 .pdb three-dimensional structure files with their assigned provenance

## How to apply

Download the 'Peptide structure.zip' package from the HORDB figshare deposit (https://doi.org/10.6084/m9.figshare.c.5522241) and extract the archive locally. Enumerate every .pdb structure file it contains, then classify each file by provenance — PDB-derived vs AlphaFold DB-derived — according to the deposit's Data Records description rather than by guessing from filenames alone. Tabulate the counts per source category in a two-column table (source category, file count) and write it to structure_file_counts.csv. The deposit's Data Records statement defines the ground truth: 54 PDB-derived and 354 AlphaFold DB-derived .pdb files, 408 in total; use this as the validation target for the classification.

## Related tools

- **HORDB** (Source of the 'Peptide structure.zip' deposit; its Data Records description defines the PDB vs AlphaFold DB provenance split (54 vs 354) used as the classification ground truth) — CPU-HORDB/HORDB

## Examples

```
unzip Peptide_structure.zip -d peptide_structures && find peptide_structures -name '*.pdb' | wc -l
```

## Evaluation signals

- Total enumerated .pdb count equals 408, matching the Data Records statement for 'Peptide structure.zip'.
- PDB-derived count equals 54 and AlphaFold DB-derived count equals 354.
- structure_file_counts.csv parses as a table with exactly the columns 'source category' and 'file count'.
- Every .pdb file in the extracted archive is assigned to exactly one provenance category (no unclassified files).
- Any mismatch between tabulated counts and the 54/354/408 Data Records figures is explicitly reported in the output table.

## Limitations

- Classification depends on the deposit's Data Records description as the authoritative provenance source; filename-only heuristics may misassign files.
- Counts are frozen to the deposit as published in 2022; later updates to PDB entries or AlphaFold DB models are not reflected.
- Only .pdb-format files are inventoried; other structure formats, if present, are out of scope.

## Evidence

- [other] Data Records enumeration workflow: [evidence span withheld — anchor status: not_found]
- [other] Expected count validation: [evidence span withheld — anchor status: not_found]
- [abstract] Deposit location: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [intro] Dataset availability: "/datasets/ Contains all datasets from HORDB"
