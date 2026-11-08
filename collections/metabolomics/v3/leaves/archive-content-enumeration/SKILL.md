---
name: archive-content-enumeration
description: 'Use when a task requires verifying the exact contents of a deposited
  data archive — e.g. confirming a stated file total (408 .pdb files: 54 PDB-derived,
  354 AlphaFold DB-derived) against the deposit itself.'
when_to_use_negative:
- The analysis target is the HORDB entry-level data (6024 entries, sequences, activity
  fields like IC50/EC50) rather than the 3D-structure file inventory.
- Structure files are already enumerated and classified, e.g. an existing per-source
  feature table — recounting adds no information.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: manual expert review
  role: Classifying each .pdb file by provenance (PDB-derived vs AlphaFold DB-derived)
    and validating tabulated counts against the Data Records statement
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:7859bf93afaf6feaa447726a96885e4d802db2d6273b9415ae19104f0a83e0f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# archive-content-enumeration

## Summary

A reproducible procedure for downloading a data-deposit archive (e.g. the HORDB 'Peptide structure.zip' package from figshare), extracting it, enumerating its .pdb structure files, and classifying them by provenance to produce auditable per-source file counts.

## When to use

Use when a task requires verifying the exact contents of a deposited data archive — e.g. confirming a stated file total (408 .pdb files: 54 PDB-derived, 354 AlphaFold DB-derived) against the deposit itself.

## When NOT to use

- The analysis target is the HORDB entry-level data (6024 entries, sequences, activity fields like IC50/EC50) rather than the 3D-structure file inventory.
- Structure files are already enumerated and classified, e.g. an existing per-source feature table — recounting adds no information.

## Inputs

- figshare deposit archive URL (https://doi.org/10.6084/m9.figshare.c.5522241)
- 'Peptide structure.zip' archive of three-dimensional peptide-hormone structure files
- Data Records section of the manuscript describing per-source file counts

## Outputs

- structure_file_counts.csv with columns: source category, file count
- Mismatch report if enumerated counts differ from the stated 54 PDB / 354 AlphaFold DB / 408 total

## How to apply

Download the archive in question — for HORDB this is the 'Peptide structure.zip' package hosted at the figshare deposit https://doi.org/10.6084/m9.figshare.c.5522241. Extract the archive locally and enumerate all files with the .pdb extension, recording their paths. Classify each .pdb file by provenance (PDB-derived vs AlphaFold DB-derived) using the deposit's Data Records description as the authority. Tabulate counts per source category in a CSV table with columns for source category and file count, then reconcile the tabulated counts against the manuscript's stated numbers (54 PDB + 354 AlphaFold DB = 408 total) and flag any mismatch explicitly in the output table. Manual expert review is the designated tool for classification and validation.

## Related tools

- **manual expert review** (Classifying each .pdb file by provenance (PDB-derived vs AlphaFold DB-derived) and validating tabulated counts against the Data Records statement)

## Examples

```
curl -L -o peptide_structure.zip https://doi.org/10.6084/m9.figshare.c.5522241 && unzip peptide_structure.zip && find . -name '*.pdb' | wc -l
```

## Evaluation signals

- Total .pdb count in the archive equals 408.
- PDB-derived count equals 54 and AlphaFold DB-derived count equals 354.
- structure_file_counts.csv contains both required columns (source category, file count) and no duplicate categories.
- Every .pdb file in the extracted archive is assigned to exactly one provenance category.
- Any deviation from the stated counts is explicitly reported in the output table rather than silently ignored.

## Limitations

- Classification depends on the deposit's Data Records description; if a .pdb file's provenance is ambiguous, it cannot be assigned without external evidence.
- Counts are only valid for the 'Peptide structure.zip' package in the c.5522241 figshare collection; future deposit revisions may change the inventory.
- The workflow is a manual expert-review process, so it does not scale to very large archives without automation.

## Evidence

- [abstract] All data are hosted and available in figshare: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] datasets directory in repo: "/datasets/ Contains all datasets from HORDB"
- [readme] download page availability: "All datasets are also available on HORDB's download page http://hordb.cpu-bioinfor.org/"
