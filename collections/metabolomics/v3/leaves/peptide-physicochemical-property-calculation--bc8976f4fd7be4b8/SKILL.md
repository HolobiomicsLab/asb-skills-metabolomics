---
name: peptide-physicochemical-property-calculation
description: Use when you have peptide hormone sequences in FASTA format from HORDB
  (figshare 10.6084/m9.figshare.c.5522241) and need to reproduce or audit net charge,
  hydrophobic fraction, and length, or characterize hydrophobicity profiles (e.g.
  the 20-40% hydrophobic residue range).
when_to_use_negative:
- Input sequences are non-peptide (e.g. nucleotide records or small-molecule drug
  entries) — the parameter module assumes amino-acid FASTA from HORDB's 5729 peptide
  hormones
- You only need to look up stored physical information rather than recompute it —
  query the HORDB database at http://hordb.cpu-bioinfor.org/ directly instead
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SciDBMaker
  role: Physicochemical-parameter module used to compute net charge, hydrophobic residue
    fraction, and sequence length for each peptide hormone sequence
- name: pandas
  role: Assembles the per-peptide computed values into a table, joins against 'Peptide
    Physical Information.xlsx' on peptide identifier, and exports peptide_physicochemical_properties.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:d9e93c06d5c598d8bd172eae031a9ed4ee57617008c8899de22d6e41f645c856
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciDBMaker
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# peptide-physicochemical-property-calculation

## Summary

Recomputes and validates the per-peptide physicochemical parameters (net charge, hydrophobic residue fraction, sequence length) underlying HORDB's peptide hormone physical information using SciDBMaker plus pandas, reconciling computed values against 'Peptide Physical Information.xlsx'.

## When to use

Use when you have peptide hormone sequences in FASTA format from HORDB (figshare 10.6084/m9.figshare.c.5522241) and need to reproduce or audit net charge, hydrophobic fraction, and length, or characterize hydrophobicity profiles (e.g. the 20-40% hydrophobic residue range).

## When NOT to use

- Input sequences are non-peptide (e.g. nucleotide records or small-molecule drug entries) — the parameter module assumes amino-acid FASTA from HORDB's 5729 peptide hormones
- You only need to look up stored physical information rather than recompute it — query the HORDB database at http://hordb.cpu-bioinfor.org/ directly instead

## Inputs

- HORDB peptide hormone FASTA files from the figshare collection https://doi.org/10.6084/m9.figshare.c.5522241
- 'Peptide Physical Information.xlsx' reference spreadsheet from the HORDB deposit

## Outputs

- peptide_physicochemical_properties.csv — reconciled per-peptide table of net charge, hydrophobic residue fraction, and sequence length with mismatch flags

## How to apply

Download the HORDB peptide FASTA files from the figshare collection (https://doi.org/10.6084/m9.figshare.c.5522241) and parse each record for sequence and identifier. Run the SciDBMaker physicochemical-parameter module on every sequence: net charge as the sum of charged-residue contributions at physiological pH, hydrophobic residue fraction as count of hydrophobic residues divided by sequence length, and length as the amino-acid count. Assemble per-peptide values with pandas into a table indexed by peptide identifier, then join to 'Peptide Physical Information.xlsx' from the HORDB deposit on peptide identifier. Flag mismatches beyond rounding tolerance; the article's reported result is that most peptide hormones fall in a 20-40% hydrophobic residue proportion, which is the expected hydrophobicity profile on success.

## Related tools

- **SciDBMaker** (Physicochemical-parameter module used to compute net charge, hydrophobic residue fraction, and sequence length for each peptide hormone sequence)
- **pandas** (Assembles the per-peptide computed values into a table, joins against 'Peptide Physical Information.xlsx' on peptide identifier, and exports peptide_physicochemical_properties.csv)

## Examples

```
python -c "import pandas as pd; comp = pd.read_csv('computed_params.csv'); ref = pd.read_excel('Peptide Physical Information.xlsx'); comp.merge(ref, on='peptide_id').to_csv('peptide_physicochemical_properties.csv', index=False)"
```

## Evaluation signals

- Join against 'Peptide Physical Information.xlsx' succeeds on peptide identifier for the expected record count (HORDB has 5729 peptide hormones among 6024 entries)
- Computed net charge, hydrophobic fraction, and length match reference spreadsheet values within rounding tolerance, with mismatches flagged rather than silently dropped
- Distribution of computed hydrophobic residue fractions is concentrated in the reported 20-40% range for most peptide hormones
- Output CSV contains the three computed columns (net charge, hydrophobic fraction, length) indexed by peptide identifier with no unparsed FASTA records

## Limitations

- Net charge is approximated as the sum of charged-residue contributions at physiological pH, so termini and pKa edge cases may deviate from other calculators; only rounding-tolerance agreement with the HORDB spreadsheet is expected
- Validation depends on matching identifiers between the recomputed table and 'Peptide Physical Information.xlsx'; unmatched or renamed identifiers will surface as spurious mismatches
- The hydrophobic residue fraction is a simple residue-count ratio (hydrophobic residues / sequence length), not a scale-based hydrophobicity score, so it is coarse for amphipathic peptides

## Evidence

- [abstract] abstract: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] readme: "Source code for the HORDB manuscript submitted to Sci Data for all the code to build HORDB."
