---
name: histogram-visualization-of-sequence-features
description: Use when you have a set of protein/peptide sequences (e.g. 'Peptide sequence.fasta.txt'
  from the HORDB figshare deposit) and need to characterize and visualize the distribution
  of a per-sequence numeric feature such as amino-acid length.
when_to_use_negative:
- The analysis goal is a summary statistic or outlier listing rather than a visual
  distribution — a frequency table alone suffices.
- The per-sequence feature requires external annotation (e.g. IC50, EC50, target,
  blood-brain barrier crossing info from HORDB activity fields) rather than a value
  computable directly from the sequence string.
- Input records are not sequences (e.g. HORDB's 255 marketed pharmaceutical preparation
  entries with no peptide sequence) — length computation is undefined.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0078
- http://edamontology.org/topic_3314
tools:
- name: Python
  role: 'Implementation language: Biopython SeqIO for FASTA parsing, string length
    computation, pandas for the frequency table, matplotlib for the histogram'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:4686d8170606469b909eaf7daef13d741700c7237436c37da964ef66b9bbd02d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0078
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# histogram-visualization-of-sequence-features

## Summary

Computes a numeric feature of every sequence in a FASTA file (e.g. amino-acid length of HORDB peptide hormones), aggregates it into a frequency table, and plots it as a histogram to reproduce distribution figures such as Fig. 10 showing short (~10 aa) peptides dominate.

## When to use

Use when you have a set of protein/peptide sequences (e.g. 'Peptide sequence.fasta.txt' from the HORDB figshare deposit) and need to characterize and visualize the distribution of a per-sequence numeric feature such as amino-acid length.

## When NOT to use

- The analysis goal is a summary statistic or outlier listing rather than a visual distribution — a frequency table alone suffices.
- The per-sequence feature requires external annotation (e.g. IC50, EC50, target, blood-brain barrier crossing info from HORDB activity fields) rather than a value computable directly from the sequence string.
- Input records are not sequences (e.g. HORDB's 255 marketed pharmaceutical preparation entries with no peptide sequence) — length computation is undefined.

## Inputs

- FASTA file of peptide/protein sequences (e.g. 'Peptide sequence.fasta.txt' from figshare deposit 10.6084/m9.figshare.c.5522241)

## Outputs

- Frequency table of per-sequence feature values (e.g. pandas DataFrame of amino-acid lengths)
- Histogram image (e.g. peptide_length_distribution.png) with sequence length on x-axis and peptide count on y-axis

## How to apply

Download and parse the sequence FASTA ('Peptide sequence.fasta.txt' from the HORDB figshare deposit https://doi.org/10.6084/m9.figshare.c.5522241) with Biopython's SeqIO to extract each peptide hormone sequence. Compute the amino-acid length of every sequence with Python string length operations, then aggregate lengths into a frequency table using a pandas DataFrame. Plot the distribution as a matplotlib histogram with sequence length on the x-axis and peptide count on the y-axis, mirroring Fig. 10 of the HORDB paper, and save the figure (e.g. peptide_length_distribution.png). The validation criterion is that the histogram's modal length bin falls at approximately 10 amino acids, matching the short-peptide-favored pattern reported in Fig. 10; if the mode differs, flag the output for expert_review rather than accepting it.

## Related tools

- **Python** (Implementation language: Biopython SeqIO for FASTA parsing, string length computation, pandas for the frequency table, matplotlib for the histogram)

## Examples

```
from Bio import SeqIO; import matplotlib.pyplot as plt; lens=[len(r.seq) for r in SeqIO.parse('Peptide sequence.fasta.txt','fasta')]; plt.hist(lens, bins=range(0,max(lens)+2)); plt.savefig('peptide_length_distribution.png')
```

## Evaluation signals

- Every sequence in the input FASTA contributes exactly one length value (count matches record count, e.g. the 5729 peptide hormones in HORDB).
- The modal length bin of the histogram falls at approximately 10 amino acids, matching Fig. 10 of the HORDB paper.
- The saved figure (peptide_length_distribution.png) axes are labeled: sequence length (x) vs peptide count (y).
- Histogram bin widths are uniform and integer-aligned so amino-acid counts are unambiguous.
- If the modal bin differs from ~10 aa, the output is flagged for expert_review instead of being accepted.

## Limitations

- Validation depends on the reference figure (Fig. 10); if the input FASTA is a subset of HORDB's 5729 peptide hormones, the modal bin may shift and should not be auto-reconciled to the paper's mode.
- Only features computable directly from the sequence string (e.g. length) are covered; richer per-entry features (IC50, EC50, ED50, target, BBB crossing) require joining HORDB entry tables, not this skill.

## Evidence

- [other] Computing sequence lengths from 'Peptide sequence.fasta.txt' in the HORDB figshare deposit confirms that among the peptide sequences, most peptide hormones are about 10 amino acids in length, as reported in Fig. 10.: [evidence span withheld — anchor status: not_found]
- [other] Validation: confirm the histogram's modal length bin falls at approximately 10 amino acids, matching the short-peptide-favored pattern reported in Fig. 10 of the HORDB paper; flag the output for expert_review if the mode differs.: [evidence span withheld — anchor status: not_found]
- [abstract] The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information.: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical"
- [abstract] All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 .: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241"
