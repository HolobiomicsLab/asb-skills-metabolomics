---
name: antibody-sequence-coverage-accuracy-calculation
description: Use when you have assembled contigs from de novo peptide predictions
  for monoclonal antibody chains and need to quantify sequence coverage and accuracy
  against known light/heavy chain references, including CDR regions.
when_to_use_negative:
- You only have raw MS/MS spectra without de novo peptide predictions — assembly and
  coverage calculation presuppose confidence-filtered predictions from tools like
  Casanovo or Novor
- Your goal is database-dependent identification of antibody sequences rather than
  database-independent assembly evaluation (e.g. Supernovo-style workflows with database
  search and in silico genetic recombination)
- You need to discriminate isoleucine/leucine or resolve Q vs AG and deamidated N
  vs D — these cannot be resolved via MS/MS even with higher-resolution spectra and
  tools
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2830
tools:
- name: ALPS
  role: De Bruijn assembler that stitches confidence-filtered de novo peptide predictions
    into antibody contigs (k-mer size 7) prior to coverage/accuracy computation
- name: BLAST
  role: Aligns assembled target contigs to ground-truth antibody sequences to determine
    which target (sub)sequences are covered
- name: Casanovo
  role: Deep-learning de novo sequencing tool whose predictions yield the benchmark
    97.69–99.53% light-chain coverage and 94.47–95.26% accuracy when assembled with
    ALPS
- name: Novor
  role: Spectrum-graph-based de novo sequencing tool whose predictions are an alternative
    input to the same assembly-and-alignment evaluation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:058290ede9b684cc3d50a4ce5ab48fa386791651dd0419ab6bf3b26efc4b4f7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - BLAST
  - Casanovo
  - Novor
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# antibody-sequence-coverage-accuracy-calculation

## Summary

Computes per-chain protein sequence coverage and accuracy of de novo antibody sequencing assemblies by aligning assembled contigs (e.g. ALPS output from Casanovo/Novor predictions) to ground-truth antibody sequences with BLAST, benchmarked against reported values for light and heavy chains.

## When to use

Use when you have assembled contigs from de novo peptide predictions for monoclonal antibody chains and need to quantify sequence coverage and accuracy against known light/heavy chain references, including CDR regions.

## When NOT to use

- You only have raw MS/MS spectra without de novo peptide predictions — assembly and coverage calculation presuppose confidence-filtered predictions from tools like Casanovo or Novor
- Your goal is database-dependent identification of antibody sequences rather than database-independent assembly evaluation (e.g. Supernovo-style workflows with database search and in silico genetic recombination)
- You need to discriminate isoleucine/leucine or resolve Q vs AG and deamidated N vs D — these cannot be resolved via MS/MS even with higher-resolution spectra and tools

## Inputs

- Confidence-filtered de novo peptide predictions (per tool, at confidence-score thresholds where amino-acid-level precision falls below 50%)
- Ground-truth antibody light- and heavy-chain reference sequences
- Reference CDR sequences from the Natural Antibody database
- Assembled contigs from ALPS (k-mer size 7)

## Outputs

- Per-chain protein sequence coverage percentages (light and heavy chains)
- Per-chain sequence accuracy percentages
- Longest constructed contig length per chain (e.g. light chains 210–219 AAs, heavy chains >440 AAs)
- Per-CDR coverage/accuracy assessment for variable regions

## How to apply

Load confidence-filtered de novo peptide predictions (filtered at each tool's confidence-score threshold where amino-acid-level precision falls below 50%) and assemble them with the de Bruijn assembler ALPS using a k-mer size of 7, which balances sequence coverage against contig repetitiveness. Align assembled target contigs to the ground-truth antibody sequences using BLAST, treating a target (sub)sequence as covered when a contig aligns to it. Compute per-chain coverage and accuracy from the BLAST alignments, recording the longest constructed contig per chain (light chains 210–219 AAs, heavy chains >440 AAs). Additionally evaluate assembly of functionally important variable regions and CDRs by identifying reference CDR sequences from the Natural Antibody database. Validate against reported benchmarks: Casanovo-based light-chain coverage must fall within 97.69–99.53% (IgG1 to Herceptin) with 94.47–95.26% accuracy, and Casanovo-based heavy-chain coverage should reproduce the lower 76.39–93.72% range.

## Related tools

- **ALPS** (De Bruijn assembler that stitches confidence-filtered de novo peptide predictions into antibody contigs (k-mer size 7) prior to coverage/accuracy computation)
- **BLAST** (Aligns assembled target contigs to ground-truth antibody sequences to determine which target (sub)sequences are covered)
- **Casanovo** (Deep-learning de novo sequencing tool whose predictions yield the benchmark 97.69–99.53% light-chain coverage and 94.47–95.26% accuracy when assembled with ALPS)
- **Novor** (Spectrum-graph-based de novo sequencing tool whose predictions are an alternative input to the same assembly-and-alignment evaluation)

## Evaluation signals

- Casanovo-based light-chain coverage falls within the reported 97.69–99.53% range with 94.47–95.26% accuracy (Table 4)
- Casanovo-based heavy-chain coverage reproduces the lower reported 76.39–93.72% range
- Longest constructed contig lengths fall in the expected ranges: 210–219 AAs for light chains and >440 AAs for heavy chains
- Every target (sub)sequence counted as covered is supported by a BLAST alignment of an assembled contig
- CDR coverage is assessed against reference CDR sequences retrieved from the Natural Antibody database

## Limitations

- Low sequence coverage and accuracy on heavy chains demonstrate that complete de novo protein sequencing remains challenging — heavy-chain coverage (76.39–93.72% with Casanovo) is markedly lower than light-chain coverage
- Database-independent full-length protein assembly is limited even when using multiple contigs and different de novo sequencing tools (Table 4)
- Isoleucine/leucine discrimination cannot be achieved via MS/MS and requires MS3 fragmentation, capping achievable accuracy
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D)
- Missing fragmentation sites, noisy spectra and long peptide sequences pose a limit for all de novo sequencing tools

## Evidence

- [other] Casanovo predictions yielded 97.69% (IgG1) to 99.53% (Herceptin) sequence coverage of the whole antibody light chain with an accuracy of 94.47-95.26%.: [evidence span withheld — anchor status: not_found]
- [abstract] We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo.: "We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo."
- [abstract] However, low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics: "However, low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics"
- [intro] To compare the ability of previously mentioned tools to reconstruct full-length protein sequences without additional database algorithms, we employed the de Bruijn assembler ALPS: "To compare the ability of previously mentioned tools to reconstruct full-length protein sequences without additional database algorithms, we employed the de Bruijn assembler ALPS"
- [discussion] Database-independent assembly of light chains can be achieved up to a sequence coverage of 99.53% by using the de Bruijn assembler ALPS together with de novo peptide predictions from Casanovo: "Database-independent assembly of light chains can be achieved up to a sequence coverage of 99.53% by using the de Bruijn assembler ALPS together with de novo peptide predictions from Casanovo"
- [discussion] Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS but require MS3 fragmentation: "Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS but require MS3 fragmentation"
