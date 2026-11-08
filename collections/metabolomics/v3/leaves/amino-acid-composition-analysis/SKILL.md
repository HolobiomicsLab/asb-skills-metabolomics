---
name: amino-acid-composition-analysis
description: Use when you have a FASTA collection of peptide sequences and need the
  overall residue frequency distribution to determine which amino acids dominate,
  e.g., characterizing natural peptide hormone composition.
when_to_use_negative:
- The goal is per-sequence physicochemical characterization (e.g., hydrophobic residue
  percentage per peptide, Fig. 11 histogram) rather than the aggregate residue distribution.
- The input contains non-peptide sequences or heavy use of non-standard/gap characters
  that would distort a count restricted to the 20 standard amino acids.
- The analysis requires positional or motif-level composition (e.g., residue enrichment
  at peptide termini) rather than the whole-collection frequency profile.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0078
tools:
- name: Biopython (SeqIO)
  role: Parse the HORDB FASTA peptide records into sequence objects
- name: pandas
  role: Tabulate the amino-acid frequency distribution and write amino_acid_frequencies.csv
- name: matplotlib
  role: Render the residue frequency bar plot (Fig. 12 style) as amino_acid_composition.png
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41597-022-01287-5
    title: HORDB a comprehensive database of peptide hormones
  source_content_hash: sha256:cd07871cc33a57c37458a34401883f11b9c26628e2c50765dad38f4bf237a7f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biopython
  - Biopython (SeqIO)
  - Python
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0078
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# amino-acid-composition-analysis

## Summary

Computes the frequency distribution of the 20 standard amino acids across a peptide sequence collection (e.g., HORDB peptide hormone FASTA records) to identify the dominant residues — arginine, glycine, leucine and serine for natural peptide hormones, as reported in Fig. 12 of the HORDB paper.

## When to use

Use when you have a FASTA collection of peptide sequences and need the overall residue frequency distribution to determine which amino acids dominate, e.g., characterizing natural peptide hormone composition.

## When NOT to use

- The goal is per-sequence physicochemical characterization (e.g., hydrophobic residue percentage per peptide, Fig. 11 histogram) rather than the aggregate residue distribution.
- The input contains non-peptide sequences or heavy use of non-standard/gap characters that would distort a count restricted to the 20 standard amino acids.
- The analysis requires positional or motif-level composition (e.g., residue enrichment at peptide termini) rather than the whole-collection frequency profile.

## Inputs

- FASTA file(s) of peptide hormone sequences from the HORDB figshare deposit 10.6084/m9.figshare.c.5522241 (e.g., 'Peptide sequence.fasta.txt')

## Outputs

- amino_acid_frequencies.csv — frequency table of all 20 standard amino acids, descending order, values summing to ~1.0
- amino_acid_composition.png — bar plot of the residue frequency distribution in the style of Fig. 12

## How to apply

Download the HORDB sequence collection from the figshare deposit (10.6084/m9.figshare.c.5522241) and locate the FASTA file(s) such as 'Peptide sequence.fasta.txt'. Parse the records with Biopython's SeqIO and concatenate all peptide sequences into a single residue stream, ignoring gaps and non-standard characters so only the 20 standard amino acids are counted. Count occurrences of each standard amino acid and normalize by total residue count to obtain a frequency distribution, then sort residues in descending order of frequency. Tabulate with pandas into amino_acid_frequencies.csv and render the distribution as a matplotlib bar plot in the style of Fig. 12, saved as amino_acid_composition.png. The method is correct when the composition matches the HORDB paper's finding that arginine, glycine, leucine and serine are the main residues of natural peptide hormones.

## Related tools

- **Biopython (SeqIO)** (Parse the HORDB FASTA peptide records into sequence objects)
- **pandas** (Tabulate the amino-acid frequency distribution and write amino_acid_frequencies.csv)
- **matplotlib** (Render the residue frequency bar plot (Fig. 12 style) as amino_acid_composition.png)

## Examples

```
from Bio import SeqIO; seqs = [str(r.seq) for r in SeqIO.parse('Peptide sequence.fasta.txt', 'fasta')]; from collections import Counter; c = Counter(a for s in seqs for a in s if a in 'ACDEFGHIKLMNPQRSTVWY'); freq = {k: v/sum(c.values()) for k, v in c.items()}
```

## Evaluation signals

- The frequency table contains all 20 standard amino acids and their values sum to approximately 1.0.
- Arginine, glycine, leucine and serine rank among the top residues of the sorted distribution.
- The computed composition is consistent with the profile reported in Fig. 12 of the HORDB paper.
- Gap characters and non-standard residues were excluded from the residue stream (counts reflect only the 20 standard amino acids).
- The bar plot amino_acid_composition.png visually matches the Fig. 12 style and ordering (descending frequency).

## Limitations

- Non-standard characters and gaps are deliberately discarded; sequences dominated by such characters will contribute fewer residues to the distribution.
- The reported dominant residues (Arg, Gly, Leu, Ser) are specific to the natural peptide hormones in HORDB (5729 entries); other peptide corpora may yield different top residues.
- The HORDB download page (http://hordb.cpu-bioinfor.org/) and figshare deposit are the authoritative sources; stale or partial FASTA files will bias the composition.

## Evidence

- [results] The amino acid distribution analysis of HORDB peptide hormone sequences identifies arginine, glycine, leucine and serine as the main residues of natural peptide hormones.: [evidence span withheld — anchor status: not_found]
- [abstract] All data are hosted and available in figshare: "All data are hosted and available in figshare https://doi.org/10.6084/m9.figshare.c.5522241 ."
- [readme] datasets directory in README: "/datasets/ Contains all datasets from HORDB"
