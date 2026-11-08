---
name: peptide-database-search-execution
description: Use when MGF-converted MS/MS spectra (e.g. enzymatic antibody digests)
  need reference peptide identifications from a known protein database, to serve as
  pseudo-ground truth or benchmark labels for downstream tool evaluation.
when_to_use_negative:
- The analysis goal is de novo sequencing itself or database-independent assembly
  (e.g. ALPS contig building) with no reference sequences available.
- The enzyme and hence cleavage parameters for the input files are unknown, since
  SearchGUI settings must be chosen enzyme-specifically.
- Spectra are not searched against a trusted reference database — this skill produces
  database-dependent identifications, not de novo predictions.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS-GF+
  role: First database search engine executed via SearchGUI on the antibody MS/MS
    data
- name: X!Tandem
  role: Second database search engine combined with MS-GF+ results to broaden identification
    coverage
- name: SearchGUI
  role: Orchestrates and parameterizes both search engines (v4.1.7), with enzyme-specific
    cleavage parameters per input file
- name: PeptideShaker
  role: Consolidates PSMs from both engines and applies the 1% FDR filter (v2.2.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:86bfc9d9bef2bf0b457009ecbe49d8ce9997ec1e796809259f9dad92cbb851a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - Novor
  - PeptideShaker
  - SearchGUI
  - X!Tandem
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# peptide-database-search-execution

## Summary

Generates high-confidence peptide identifications by running two complementary search engines (MS-GF+ and X!Tandem) through SearchGUI against a reference protein database, then consolidating and FDR-filtering PSMs with PeptideShaker — used here to build the 23,844-peptide pseudo-ground truth for evaluating de novo sequencing tools.

## When to use

Use when MGF-converted MS/MS spectra (e.g. enzymatic antibody digests) need reference peptide identifications from a known protein database, to serve as pseudo-ground truth or benchmark labels for downstream tool evaluation.

## When NOT to use

- The analysis goal is de novo sequencing itself or database-independent assembly (e.g. ALPS contig building) with no reference sequences available.
- The enzyme and hence cleavage parameters for the input files are unknown, since SearchGUI settings must be chosen enzyme-specifically.
- Spectra are not searched against a trusted reference database — this skill produces database-dependent identifications, not de novo predictions.

## Inputs

- MGF-converted MS/MS spectra files from enzymatic antibody digests
- Antibody reference sequences used as the protein database (e.g. FASTA)
- Per-input-file enzyme cleavage specification

## Outputs

- High-confidence PSM list filtered at 1% FDR from PeptideShaker
- Reference set of 23,844 database-search-identified peptides across three antibody data sets (pseudo-ground truth)

## How to apply

Load the MGF-converted MS/MS experiments and use the antibody reference sequences as the protein search database. Execute MS-GF+ and X!Tandem jointly through SearchGUI (v4.1.7), choosing cleavage parameters per input file according to each file's digestion enzyme, with a precursor tolerance of 10 ppm and fragment mass tolerance of 0.02 Da; set carbamidomethylation of cysteine (C +57.02 Da) as a fixed modification and oxidation of methionine (M +15.99 Da) plus deamidation of N and Q (+0.98 Da) as variable modifications. Post-process the combined engine results with PeptideShaker (v2.2.2) to consolidate PSMs, then filter all PSMs at a 1% false discovery rate to retain high-confidence identifications. Rationale: combining two engines and enforcing a strict 1% FDR yields a reference set robust enough to score de novo predictions against.

## Related tools

- **MS-GF+** (First database search engine executed via SearchGUI on the antibody MS/MS data)
- **X!Tandem** (Second database search engine combined with MS-GF+ results to broaden identification coverage)
- **SearchGUI** (Orchestrates and parameterizes both search engines (v4.1.7), with enzyme-specific cleavage parameters per input file)
- **PeptideShaker** (Consolidates PSMs from both engines and applies the 1% FDR filter (v2.2.2))

## Evaluation signals

- The PeptideShaker output must contain PSMs filtered at exactly 1% FDR.
- The filtered reference set must total 23,844 peptides across the three antibody data sets.
- Both MS-GF+ and X!Tandem results must be combined (not a single-engine search) before PeptideShaker post-processing.
- Search parameters must include 10 ppm precursor tolerance, 0.02 Da fragment tolerance, fixed C +57.02 Da, and variable M +15.99 Da and N/Q +0.98 Da modifications.
- Cleavage parameters must match the declared enzyme of each input file rather than a single global setting.

## Limitations

- Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS, so database-search identifications cannot resolve I/L positions.
- Particular amino acids cannot be resolved even with higher-resolution spectra (e.g. I & L; Q & AG; deamidated N & D), affecting ground-truth precision.
- Ground truth is only as good as the antibody reference database supplied; enzyme-specific cleavage parameters must be maintained per input file.

## Evidence

- [methods] The pseudo-ground truth was produced by combining the database search results of MS-GF+ and X!Tandem, both executed via SearchGUI (v.4.1.7) and post-processed via PeptideShaker (v.2.2.2): [evidence span withheld — anchor status: not_found]
- [methods] Filter all resulting PSMs at a 1% false discovery rate to retain high-confidence identifications, yielding the reference set of 23,844 peptides across the three antibody data sets.: [evidence span withheld — anchor status: not_found]
- [abstract] De novo protein sequencing based on mass spectrometry is a valuable method to obtain the amino acid sequence of peptides and proteins without a priori knowledge.: "De novo protein sequencing based on mass spectrometry is a valuable method to obtain the amino acid sequence of peptides and proteins without a priori knowledge."
