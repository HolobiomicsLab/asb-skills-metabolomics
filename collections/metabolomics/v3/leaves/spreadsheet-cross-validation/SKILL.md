---
name: spreadsheet-cross-validation
description: Use when a deposited Excel/CSV reference table (e.g. HORDB's 'Peptide
  Physical Information.xlsx') must be verified against independently recomputed values
  from raw peptide FASTA sequences.
when_to_use_negative:
- Input is already a reconciled feature/parameter table with no raw FASTA sequences
  available for independent recomputation
- The goal is activity- or structure-level validation (IC50, EC50, ED50, target, blood-brain
  barrier fields) rather than the physical-information columns
- Sequences are not peptide hormones from HORDB and no deposited spreadsheet exists
  to join against — plain SciDBMaker computation suffices without cross-validation
tools:
- name: SciDBMaker
  role: Computes per-sequence physicochemical parameters (net charge at physiological
    pH, hydrophobic residue fraction, sequence length) for each HORDB peptide FASTA
    record
- name: pandas
  role: Parses the FASTA records into a table, loads 'Peptide Physical Information.xlsx',
    joins on peptide identifier, and writes the reconciled CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:f13f65ce60ec16c453c481162c923712b424b71bbca271b65e1a7063b24b2d5f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciDBMaker
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# spreadsheet-cross-validation

## Summary

Recomputes per-peptide physicochemical parameters (net charge, hydrophobic residue fraction, sequence length) with SciDBMaker on HORDB FASTA sequences and cross-validates them against the deposited 'Peptide Physical Information.xlsx' spreadsheet, flagging mismatches beyond rounding tolerance.

## When to use

Use when a deposited Excel/CSV reference table (e.g. HORDB's 'Peptide Physical Information.xlsx') must be verified against independently recomputed values from raw peptide FASTA sequences.

## When NOT to use

- Input is already a reconciled feature/parameter table with no raw FASTA sequences available for independent recomputation
- The goal is activity- or structure-level validation (IC50, EC50, ED50, target, blood-brain barrier fields) rather than the physical-information columns
- Sequences are not peptide hormones from HORDB and no deposited spreadsheet exists to join against — plain SciDBMaker computation suffices without cross-validation

## Inputs

- Peptide FASTA files from the HORDB figshare collection (https://doi.org/10.6084/m9.figshare.c.5522241)
- 'Peptide Physical Information.xlsx' reference spreadsheet from the HORDB deposit

## Outputs

- peptide_physicochemical_properties.csv: per-peptide table of recomputed net charge, hydrophobic residue fraction, and length, joined to the reference spreadsheet values with mismatch flags
- List of peptide identifiers whose recomputed parameters disagree with the spreadsheet beyond rounding tolerance

## How to apply

Download the HORDB peptide FASTA files from the figshare collection (https://doi.org/10.6084/m9.figshare.c.5522241) and parse each record into an identifier and sequence. Run the SciDBMaker physicochemical-parameter module on every sequence to compute net charge (sum of charged-residue contributions at physiological pH), hydrophobic residue fraction (hydrophobic residue count divided by sequence length), and amino-acid length. Assemble the computed values into a pandas DataFrame indexed by peptide identifier, then read 'Peptide Physical Information.xlsx' with pandas and join the two tables on peptide identifier. Compare net charge, hydrophobic fraction, and length column-by-column, flagging mismatches beyond rounding tolerance rather than assuming the spreadsheet is authoritative. Write the reconciled table to peptide_physicochemical_properties.csv. The expected scientific readout of the validated hydrophobic-fraction column is that most HORDB peptide hormones fall in the 20–40% hydrophobic residue range, i.e. they are largely less hydrophobic.

## Related tools

- **SciDBMaker** (Computes per-sequence physicochemical parameters (net charge at physiological pH, hydrophobic residue fraction, sequence length) for each HORDB peptide FASTA record)
- **pandas** (Parses the FASTA records into a table, loads 'Peptide Physical Information.xlsx', joins on peptide identifier, and writes the reconciled CSV)

## Examples

```
df = pd.merge(scidbmaker_compute('hordb_peptides.fasta'), pd.read_excel('Peptide Physical Information.xlsx'), on='peptide_id'); df.to_csv('peptide_physicochemical_properties.csv', index=False)
```

## Evaluation signals

- Every peptide identifier in the FASTA files is present in the joined table with non-null computed net charge, hydrophobic fraction, and length
- Sequence length equals the raw amino-acid count of each FASTA record (exact match, no rounding expected)
- Hydrophobic fractions fall in [0, 1], and the majority cluster in the 20–40% range consistent with the published finding
- Mismatch flag count against 'Peptide Physical Information.xlsx' is zero after allowing rounding tolerance; any residual mismatches are itemized per peptide
- Output peptide_physicochemical_properties.csv loads cleanly in pandas with one row per peptide identifier and no duplicate keys

## Limitations

- Rounding in the deposited spreadsheet means small discrepancies (e.g. hydrophobic fraction) must be tolerated rather than treated as errors
- Net charge at physiological pH depends on the charged-residue contribution model in SciDBMaker; results are not transferable to other charge-calculation conventions
- The skill validates only the physical-information columns, not the activity annotations (IC50, EC50, ED50, target, blood-brain barrier) of HORDB entries

## Evidence

- [abstract] All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 .: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [abstract] Each entry provides general information, sequence, activity, structure, physical information and literature information: [evidence span withheld — anchor status: not_found]
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
