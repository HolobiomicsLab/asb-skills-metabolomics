---
name: contig-to-reference-blast-alignment
description: Use when assembled de novo contigs (e.g. from ALPS over Casanovo predictions)
  must be scored against a reference antibody sequence to compute per-chain coverage
  and amino-acid accuracy, including CDR-level evaluation.
when_to_use_negative:
- When the goal is database-search peptide identification rather than scoring database-independent
  de novo assembly of full-length antibody chains
- When no reference or ground-truth antibody sequence is available for the target,
  since coverage and accuracy are defined by contig-to-reference BLAST alignments
- When the input is raw MS/MS spectra or unfiltered de novo peptide predictions -
  these must first be sequenced (e.g. with Casanovo), confidence-filtered, and assembled
  with ALPS before BLAST-based scoring
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0160
tools:
- name: BLAST
  role: Aligns the assembled target contigs to the ground-truth antibody sequences;
    a (sub)sequence counts as covered when a contig aligns to it, and alignments feed
    the coverage and accuracy computation
- name: ALPS
  role: de Bruijn assembler that produced the contigs being scored, run with k-mer
    size 7 on confidence-filtered de novo peptide predictions
- name: Casanovo
  role: Deep-learning de novo sequencer whose confidence-filtered peptide predictions
    are the input reads that ALPS assembled into the contigs evaluated here
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:a48a1455a14ffdeb6c2a1d240b4e51ed53354be81b2b2315a4390b7f78eb53f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - BLAST
  - Casanovo
  - Novor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# contig-to-reference-blast-alignment

## Summary

Quantifies de novo antibody reconstruction quality by aligning ALPS-assembled peptide contigs to ground-truth antibody sequences with BLAST, converting alignments into per-chain sequence coverage and accuracy metrics for light and heavy chains.

## When to use

Use when assembled de novo contigs (e.g. from ALPS over Casanovo predictions) must be scored against a reference antibody sequence to compute per-chain coverage and amino-acid accuracy, including CDR-level evaluation.

## When NOT to use

- When the goal is database-search peptide identification rather than scoring database-independent de novo assembly of full-length antibody chains
- When no reference or ground-truth antibody sequence is available for the target, since coverage and accuracy are defined by contig-to-reference BLAST alignments
- When the input is raw MS/MS spectra or unfiltered de novo peptide predictions - these must first be sequenced (e.g. with Casanovo), confidence-filtered, and assembled with ALPS before BLAST-based scoring

## Inputs

- FASTA of assembled contigs produced by ALPS (k-mer size 7) from confidence-filtered de novo peptide predictions
- Ground-truth antibody reference protein sequences (light and heavy chains)
- Reference CDR sequences for each antibody retrieved from the Natural Antibody database

## Outputs

- Per-chain protein sequence coverage values (e.g. light chains 97.69-99.53%, heavy chains 76.39-93.72% with Casanovo)
- Per-chain amino-acid-level sequence accuracy (e.g. 94.47-95.26% on light chains with Casanovo)
- Longest constructed contig length per chain (light chains 210-219 AAs, heavy chains >440 AAs)
- Variable-region and CDR-level coverage assessment per antibody

## How to apply

Load the confidence-filtered de novo peptide predictions (filtered at each tool's threshold where amino-acid-level precision falls below 50%) that were assembled with the de Bruijn assembler ALPS using a k-mer size of 7, chosen to balance sequence coverage against contig repetitiveness. Align the assembled target contigs to the ground-truth antibody sequences using BLAST, treating a target (sub)sequence as covered when a contig aligns to it. From the BLAST alignments, compute per-chain protein sequence coverage and accuracy, and record the longest constructed contig for each chain (light chains 210-219 AAs, heavy chains >440 AAs). Separately evaluate assembly of the functionally important variable regions and CDRs by identifying reference CDR sequences for each antibody from the Natural Antibody database. Benchmark against the reported Casanovo+ALPS results: light-chain coverage 97.69-99.53% (IgG1 to Herceptin) with 94.47-95.26% accuracy, and lower heavy-chain coverage of 76.39-93.72%.

## Related tools

- **BLAST** (Aligns the assembled target contigs to the ground-truth antibody sequences; a (sub)sequence counts as covered when a contig aligns to it, and alignments feed the coverage and accuracy computation)
- **ALPS** (de Bruijn assembler that produced the contigs being scored, run with k-mer size 7 on confidence-filtered de novo peptide predictions)
- **Casanovo** (Deep-learning de novo sequencer whose confidence-filtered peptide predictions are the input reads that ALPS assembled into the contigs evaluated here)

## Examples

```
blastp -query alps_contigs.fasta -subject antibody_reference.fasta -out contig_alignment.tsv
```

## Evaluation signals

- Light-chain coverage using Casanovo+ALPS contigs must fall within the reported 97.69-99.53% range (IgG1 to Herceptin)
- Light-chain amino-acid accuracy must reproduce the reported 94.47-95.26% range (Table 4)
- Heavy-chain coverage should reproduce the lower reported range of 76.39-93.72% with Casanovo; substantially higher values suggest erroneous contig-to-reference alignment
- Longest contig lengths must match the reported scale: 210-219 AAs for light chains and >440 AAs for heavy chains
- A target (sub)sequence is only counted as covered when a contig aligns to it in the BLAST output - unaligned reference stretches must remain uncovered

## Limitations

- Low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics
- Database-independent full-length protein assembly is limited even when using multiple contigs and different de novo sequencing tools (Table 4)
- Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS, so assembled contigs cannot resolve I/L positions (requires MS3)
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D)

## Evidence

- [abstract] Light-chain coverage reproduced from abstract: "We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo."
- [intro] Assembly objective from intro: "To compare the ability of previously mentioned tools to reconstruct full-length protein sequences without additional database algorithms, we employed the de Bruijn assembler ALPS"
- [discussion] Assembly ceiling from discussion: "Database-independent assembly of light chains can be achieved up to a sequence coverage of 99.53% by using the de Bruijn assembler ALPS together with de novo peptide predictions from Casanovo"
- [abstract] Heavy-chain challenge from abstract: "However, low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics"
