---
name: short-peptide-statistical-characterization
description: Use when you have peptide sequences (e.g. a FASTA file from HORDB's 'Peptide
  sequence.fasta.txt') and need to characterize their sequence-length distribution,
  e.g. to verify that the modal length bin falls near ~10 amino acids.
when_to_use_negative:
- The input is not peptide hormone sequence data from HORDB (e.g. nucleotide sequences
  or non-HORDB peptide sets with different length expectations).
- The goal is to analyze activity or structural annotations (IC50, EC50, ED50, target,
  blood-brain barrier information) rather than sequence-length statistics.
- A per-sequence quality/annotation check is needed rather than an aggregate length
  distribution.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: Python
  role: Scripting environment used to parse the FASTA file (Biopython SeqIO), compute
    sequence lengths, aggregate frequencies (pandas), and plot the histogram (matplotlib).
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:fa8e34c6651e3fe300aadbe0ed58eaf4f67eb545bb00a465ebbf0db5646481a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# short-peptide-statistical-characterization

## Summary

Compute and visualize the amino-acid length distribution of peptide hormone sequences from the HORDB figshare deposit, demonstrating that most peptide hormones are short (~10 aa) as reported in Fig. 10 of the HORDB paper.

## When to use

Use when you have peptide sequences (e.g. a FASTA file from HORDB's 'Peptide sequence.fasta.txt') and need to characterize their sequence-length distribution, e.g. to verify that the modal length bin falls near ~10 amino acids.

## When NOT to use

- The input is not peptide hormone sequence data from HORDB (e.g. nucleotide sequences or non-HORDB peptide sets with different length expectations).
- The goal is to analyze activity or structural annotations (IC50, EC50, ED50, target, blood-brain barrier information) rather than sequence-length statistics.
- A per-sequence quality/annotation check is needed rather than an aggregate length distribution.

## Inputs

- 'Peptide sequence.fasta.txt' from the HORDB figshare deposit (https://doi.org/10.6084/m9.figshare.c.5522241), a FASTA-format file of peptide hormone sequences

## Outputs

- Frequency table of peptide sequence lengths (pandas DataFrame)
- Histogram of peptide length distribution saved as peptide_length_distribution.png

## How to apply

Download 'Peptide sequence.fasta.txt' from the HORDB figshare collection (https://doi.org/10.6084/m9.figshare.c.5522241) and parse each peptide hormone record with Biopython's SeqIO. Compute the amino-acid length of every sequence with plain Python string length operations, then aggregate lengths into a frequency table using a pandas DataFrame. Plot the distribution as a histogram (matplotlib) with sequence length on the x-axis and peptide count on the y-axis, mirroring Fig. 10 of the HORDB paper, and save it as peptide_length_distribution.png. The key decision point is validation: confirm the histogram's modal length bin sits at approximately 10 amino acids; if the mode differs from the short-peptide-favored pattern reported in Fig. 10, flag the output for expert_review rather than accepting it.

## Related tools

- **Python** (Scripting environment used to parse the FASTA file (Biopython SeqIO), compute sequence lengths, aggregate frequencies (pandas), and plot the histogram (matplotlib).)

## Examples

```
from Bio import SeqIO; lengths = [len(rec.seq) for rec in SeqIO.parse('Peptide sequence.fasta.txt', 'fasta')]; import pandas as pd, matplotlib.pyplot as plt; pd.Series(lengths).value_counts().sort_index().plot(kind='bar'); plt.savefig('peptide_length_distribution.png')
```

## Evaluation signals

- The histogram's modal length bin falls at approximately 10 amino acids, matching the short-peptide-dominated pattern of Fig. 10 in the HORDB paper.
- The number of parsed sequences matches the expected count of HORDB peptide hormones (5729 peptide hormones among 6024 total entries).
- Every sequence length is a positive integer consistent with amino-acid strings (no parse errors or empty records from SeqIO).
- The output figure peptide_length_distribution.png has sequence length on the x-axis and peptide count on the y-axis.
- Outputs where the mode deviates from ~10 aa are flagged for expert_review.

## Limitations

- The expected ~10 aa modal length is specific to the HORDB peptide hormone set; other peptide collections may have different distributions and would fail the validation check.
- HORDB also contains peptide drugs (40) and pharmaceutical preparations information (255), which are not part of the peptide hormone length distribution and should be excluded or interpreted carefully.

## Evidence

- [results] Computing sequence lengths from 'Peptide sequence.fasta.txt' in the HORDB figshare deposit confirms that among the peptide sequences, most peptide hormones are about 10 amino acids in length, as reported in Fig. 10.: [evidence span withheld — anchor status: not_found]
- [abstract] All data are hosted and available in figshare: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241"
- [abstract] The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information.: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information."
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
