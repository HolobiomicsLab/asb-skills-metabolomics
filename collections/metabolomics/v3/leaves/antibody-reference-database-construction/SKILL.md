---
name: antibody-reference-database-construction
description: Use when antibody amino acid sequences must be turned into a protein
  search database to generate database-search-derived pseudo-ground truth (high-confidence
  PSMs) against which de novo sequencing tool predictions on antibody MS/MS data are
  evaluated.
when_to_use_negative:
- You already have de novo sequencing predictions and want to assemble them into full-length
  antibody sequences (use the de Bruijn assembler ALPS workflow instead)
- The data are non-antibody proteomics samples where a full proteome database, not
  a small antibody-sequence database, is the appropriate search target
- You only need spectral pre-processing (e.g. MGF conversion) without any database-search-based
  reference generation
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SearchGUI
  role: Wrapper that executes both search engines (MS-GF+ and X!Tandem, v.4.1.7) with
    per-file enzyme-specific cleavage parameters
- name: MS-GF+
  role: First database search engine run against the antibody reference sequences
- name: X!Tandem
  role: Second database search engine run against the antibody reference sequences
    for combined identification
- name: PeptideShaker
  role: Post-processes and consolidates PSMs from both engines (v.2.2.2) before 1%
    FDR filtering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:00ab84eff7d2e8aa4ac91424520bf1b4329ced5128fb9c147e1a0d92fb1b1e72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - Novor
  - PeptideShaker
  - SearchGUI
  - X!Tandem
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# antibody-reference-database-construction

## Summary

Constructs the antibody protein reference database and derives a pseudo-ground-truth PSM set by running combined MS-GF+ and X!Tandem searches (via SearchGUI v.4.1.7) against antibody sequences, post-processing with PeptideShaker v.2.2.2, and filtering at 1% FDR to benchmark de novo sequencing predictions.

## When to use

Use when antibody amino acid sequences must be turned into a protein search database to generate database-search-derived pseudo-ground truth (high-confidence PSMs) against which de novo sequencing tool predictions on antibody MS/MS data are evaluated.

## When NOT to use

- You already have de novo sequencing predictions and want to assemble them into full-length antibody sequences (use the de Bruijn assembler ALPS workflow instead)
- The data are non-antibody proteomics samples where a full proteome database, not a small antibody-sequence database, is the appropriate search target
- You only need spectral pre-processing (e.g. MGF conversion) without any database-search-based reference generation

## Inputs

- MGF-converted antibody MS/MS experiment files (one per enzymatic digest)
- Antibody (monoclonal) reference amino acid sequences used as the protein database
- Per-file enzyme specification for enzyme-specific cleavage parameters

## Outputs

- Filtered PSM list at 1% FDR (pseudo-ground truth) — 23,844 database-search-identified peptides across the three antibody data sets
- Consolidated PeptideShaker (v.2.2.2) identification results combining MS-GF+ and X!Tandem matches

## How to apply

Load the MGF-converted MS/MS experiments and use the antibody reference sequences as the protein database for the search. Execute MS-GF+ and X!Tandem through SearchGUI (v.4.1.7) with cleavage parameters chosen according to the enzyme of each input file, precursor tolerance of 10 ppm, fragment mass tolerance of 0.02 Da, carbamidomethylation of cysteine (C + 57.02 Da) as a fixed modification, and oxidation of methionine (M + 15.99 Da) plus deamidation of N and Q (+0.98 Da) as variable modifications. Post-process the combined search results with PeptideShaker (v.2.2.2) to consolidate peptide-spectrum matches from both engines, then filter all resulting PSMs at a 1% false discovery rate. The rationale is that two independent search engines combined per enzyme give a defensible identification-based reference, unlike purely de novo predictions, which cannot establish ground truth for evaluation.

## Related tools

- **SearchGUI** (Wrapper that executes both search engines (MS-GF+ and X!Tandem, v.4.1.7) with per-file enzyme-specific cleavage parameters)
- **MS-GF+** (First database search engine run against the antibody reference sequences)
- **X!Tandem** (Second database search engine run against the antibody reference sequences for combined identification)
- **PeptideShaker** (Post-processes and consolidates PSMs from both engines (v.2.2.2) before 1% FDR filtering)

## Evaluation signals

- All retained PSMs pass a 1% false discovery rate filter
- Filtered output matches the reported 23,844 database-search-identified peptides across the three antibody data sets
- PSMs are consolidated from both MS-GF+ and X!Tandem via PeptideShaker, not from a single engine
- Search parameters are consistent with the specification: 10 ppm precursor tolerance, 0.02 Da fragment tolerance, fixed C + 57.02 Da, variable M + 15.99 Da and N/Q deamidation +0.98 Da
- Enzyme-specific cleavage parameters differ correctly per input file according to each digest's enzyme

## Limitations

- The pseudo-ground truth is only as good as the antibody reference sequences provided; incomplete or incorrect sequences will bias the FDR-filtered PSM set
- Combining only two search engines may miss identifications that additional engines would recover, so the reference set is not exhaustive
- The filtered reference covers identified peptides only — it does not resolve unidentifiable regions such as isoleucine/leucine discrimination, which cannot be achieved via MS/MS

## Evidence

- [other] The pseudo-ground truth was produced by combining the database search results of MS-GF+ and X!Tandem, both executed via SearchGUI (v.4.1.7) and post-processed via PeptideShaker (v.2.2.2): [evidence span withheld — anchor status: not_found]
- [other] Filter all resulting PSMs at a 1% false discovery rate to retain high-confidence identifications, yielding the reference set of 23,844 peptides across the three antibody data sets.: [evidence span withheld — anchor status: not_found]
- [abstract] We validated their ability to identify and assemble antibody sequences on three multi-enzymatic data sets.: "We validated their ability to identify and assemble antibody sequences on three multi-enzymatic data sets."
