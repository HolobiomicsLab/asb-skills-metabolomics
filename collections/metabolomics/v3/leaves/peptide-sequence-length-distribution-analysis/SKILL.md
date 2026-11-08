---
name: peptide-sequence-length-distribution-analysis
description: Use when you have a peptide FASTA file (e.g. 'Peptide sequence.fasta.txt'
  from the HORDB figshare deposit) and need to quantify or validate the sequence-length
  distribution of peptide hormones against the ~10 aa mode reported in Fig. 10.
when_to_use_negative:
- Your goal is classifying or annotating peptide sequences (activity, targets, IC50/EC50)
  rather than characterizing their length distribution
- The input is not peptide hormone sequences in FASTA form — e.g. nucleotide sequences
  or pharmaceutical preparation records from HORDB's 6024 entries
- You need peptide drugs or marketed pharmaceutical preparations statistics instead
  of the peptide hormone sequence-length profile
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: Biopython SeqIO
  role: Parses the FASTA peptide sequences file to extract each peptide hormone sequence
- name: pandas
  role: Aggregates per-sequence amino-acid lengths into a frequency table
- name: matplotlib
  role: Plots the length histogram mirroring Fig. 10 of the HORDB paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:8307b782d5a06cb717ef46c76f159ec086f48a591ea5dd4907e7cd6337d08c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biopython SeqIO
  - Python
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# peptide-sequence-length-distribution-analysis

## Summary

Reproduces the HORDB peptide hormone length distribution by computing amino-acid lengths of sequences in 'Peptide sequence.fasta.txt' from the figshare deposit and plotting a histogram (mirroring Fig. 10), verifying that short ~10 aa peptides dominate.

## When to use

Use when you have a peptide FASTA file (e.g. 'Peptide sequence.fasta.txt' from the HORDB figshare deposit) and need to quantify or validate the sequence-length distribution of peptide hormones against the ~10 aa mode reported in Fig. 10.

## When NOT to use

- Your goal is classifying or annotating peptide sequences (activity, targets, IC50/EC50) rather than characterizing their length distribution
- The input is not peptide hormone sequences in FASTA form — e.g. nucleotide sequences or pharmaceutical preparation records from HORDB's 6024 entries
- You need peptide drugs or marketed pharmaceutical preparations statistics instead of the peptide hormone sequence-length profile

## Inputs

- 'Peptide sequence.fasta.txt' — FASTA-formatted peptide hormone sequences from the HORDB figshare deposit (https://doi.org/10.6084/m9.figshare.c.5522241)

## Outputs

- Frequency table of peptide sequence lengths (pandas DataFrame)
- peptide_length_distribution.png — histogram of sequence length (x-axis) vs peptide count (y-axis)

## How to apply

Download 'Peptide sequence.fasta.txt' from the HORDB figshare collection (https://doi.org/10.6084/m9.figshare.c.5522241) and parse it with Biopython's SeqIO to extract each peptide hormone sequence. Compute each sequence's amino-acid length with Python string length operations, then aggregate lengths into a frequency table (e.g. a pandas DataFrame). Plot the length distribution as a matplotlib histogram with sequence length on the x-axis and peptide count on the y-axis, mirroring Fig. 10 of the HORDB paper, and save it as peptide_length_distribution.png. Validate by checking that the histogram's modal length bin falls at approximately 10 amino acids; flag the output for expert_review if the mode differs.

## Related tools

- **Biopython SeqIO** (Parses the FASTA peptide sequences file to extract each peptide hormone sequence)
- **pandas** (Aggregates per-sequence amino-acid lengths into a frequency table)
- **matplotlib** (Plots the length histogram mirroring Fig. 10 of the HORDB paper)

## Examples

```
from Bio import SeqIO; lens=[len(r.seq) for r in SeqIO.parse('Peptide sequence.fasta.txt','fasta')]
```

## Evaluation signals

- The histogram's modal length bin falls at approximately 10 amino acids, matching the short-peptide-favored pattern in Fig. 10 of the HORDB paper
- The number of parsed FASTA records is consistent with the peptide hormones in HORDB (5729 peptide hormones out of 6024 total entries)
- Every computed length is a positive integer and no sequence record is dropped silently during parsing
- Frequency-table counts sum to the number of parsed sequences
- Output figure peptide_length_distribution.png exists and axes match sequence length (x) vs peptide count (y); flag for expert_review if the mode differs from ~10 aa

## Limitations

- Validation relies on the modal bin approximating 10 aa; deviations (e.g. after re-filtering the deposit) require flagging for expert_review rather than automatic correction
- HORDB also contains 40 peptide drugs and 255 marketed pharmaceutical preparations whose lengths should not be mixed into the peptide hormone length distribution

## Evidence

- [results] Computing sequence lengths from 'Peptide sequence.fasta.txt' in the HORDB figshare deposit confirms that among the peptide sequences, most peptide hormones are about 10 amino acids in length: [evidence span withheld — anchor status: not_found]
- [abstract] All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241"
- [readme] /datasets/ Contains all datasets from HORDB: "/datasets/ Contains all datasets from HORDB"
- [abstract] The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information: "The database currently has a total of 6024 entries, including 5729 peptide hormones, 40 peptide drugs and 255 marketed pharmaceutical preparations information"
