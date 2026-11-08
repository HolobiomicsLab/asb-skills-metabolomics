---
name: enzyme-specific-cleavage-parameter-configuration
description: Use when MGF-converted antibody MS/MS files from multiple enzymatic digests
  must be searched against antibody reference sequences to generate trusted PSMs (pseudo-ground
  truth) for evaluating de novo sequencing predictions.
when_to_use_negative:
- The goal is de novo sequencing itself rather than building a database-search-derived
  reference set — in that case the de novo tools (Novor, pNovo 3, DeepNovo, SMSNet,
  PointNovo, Casanovo) are the subjects of the evaluation, not the ground truth generators.
- Input spectra are not enzyme-digest peptide MS/MS data and no enzyme-specific cleavage
  parameters can be defined for them.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SearchGUI
  role: Runs MS-GF+ and X!Tandem database searches with enzyme-specific cleavage parameters
    (v.4.1.7)
- name: MS-GF+
  role: First search engine combined in the database search against antibody sequences
- name: X!Tandem
  role: Second search engine combined in the database search against antibody sequences
- name: PeptideShaker
  role: Consolidates PSMs from both engines and applies the 1% FDR filter (v.2.2.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:eb4bd40d97b189f9d59d1dc969a483773a8fcf12abf0cf7efc399fe37e19a073
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# enzyme-specific-cleavage-parameter-configuration

## Summary

Configure database search engines (MS-GF+ and X!Tandem, run through SearchGUI) with enzyme-specific cleavage parameters matched to each input antibody digest, then consolidate and filter the combined PSMs at 1% FDR in PeptideShaker to yield a high-confidence pseudo-ground-truth peptide set for benchmarking de novo sequencing tools.

## When to use

Use when MGF-converted antibody MS/MS files from multiple enzymatic digests must be searched against antibody reference sequences to generate trusted PSMs (pseudo-ground truth) for evaluating de novo sequencing predictions.

## When NOT to use

- The goal is de novo sequencing itself rather than building a database-search-derived reference set — in that case the de novo tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) are the subjects of the evaluation, not the ground truth generators.
- Input spectra are not enzyme-digest peptide MS/MS data and no enzyme-specific cleavage parameters can be defined for them.

## Inputs

- MGF-converted MS/MS experiment files from antibody enzymatic digests
- Antibody reference sequences used as the protein database for the search

## Outputs

- High-confidence peptide-spectrum matches (PSMs) filtered at 1% FDR
- A pseudo-ground-truth peptide list (e.g. 23,844 database-search-identified peptides across three antibody data sets) for benchmarking de novo sequencing tools

## How to apply

Load the MGF-converted MS/MS experiments together with the antibody sequences as the protein database, and execute MS-GF+ and X!Tandem through SearchGUI (v.4.1.7), choosing cleavage parameters according to the enzyme of each input file. Apply a precursor tolerance of 10 ppm, a fragment mass tolerance of 0.02 Da, carbamidomethylation of cysteine (C + 57.02 Da) as a fixed modification, and oxidation of methionine (M + 15.99 Da) plus deamidation of N and Q (+0.98 Da) as variable modifications. Post-process the combined results of both engines with PeptideShaker (v.2.2.2) to consolidate peptide-spectrum matches. Finally, filter all resulting PSMs at a 1% false discovery rate to retain high-confidence identifications; the rationale is that a strict FDR cutoff on an antibody-sequence-restricted database provides a defensible pseudo-ground truth against which de novo predictions (e.g. from Novor or Casanovo) can be scored.

## Related tools

- **SearchGUI** (Runs MS-GF+ and X!Tandem database searches with enzyme-specific cleavage parameters (v.4.1.7))
- **MS-GF+** (First search engine combined in the database search against antibody sequences)
- **X!Tandem** (Second search engine combined in the database search against antibody sequences)
- **PeptideShaker** (Consolidates PSMs from both engines and applies the 1% FDR filter (v.2.2.2))

## Examples

```
searchgui --engines ms-gf+,x-tandem --input spectra.mgf --database antibody.fasta --enzyme trypsin --precursor-tolerance 10ppm --fragment-tolerance 0.02Da --fixed-mods C:57.02 --variable-mods M:15.99,N:0.98,Q:0.98
```

## Evaluation signals

- The filtered output contains PSMs at 1% FDR, matching the reported 23,844 database-search-identified peptides used as ground truth.
- Search parameters recorded include precursor tolerance 10 ppm, fragment mass tolerance 0.02 Da, fixed C + 57.02 Da, and variable M + 15.99 / N,Q + 0.98 Da.
- Cleavage parameters differ appropriately per input file, matching the enzyme of each antibody digest.
- PeptideShaker output consolidates identifications from both MS-GF+ and X!Tandem rather than reporting one engine alone.

## Limitations

- Ground truth is only as good as the antibody-sequence database; identifications are restricted to known antibody reference sequences.
- Isoleucine and leucine cannot be discriminated via MS/MS, so the ground truth cannot resolve these isomeric residues.
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D), affecting pseudo-ground-truth quality.
- Enzyme specificity limits comparison: DL tools applied to non-tryptic data will show performance that varies with the cleavage pattern of their trained data set.

## Evidence

- [methods] Pseudo-ground truth via combined database search at 1% FDR: [evidence span withheld — anchor status: not_found]
- [intro] Enzymatic digest design motivates per-file cleavage parameters: "In contrast to previous studies, we evaluate de novo peptide sequencing tools on various enzymatic antibody data sets."
- [discussion] Enzyme-specific handling is a known demanding task: "training different models for over six enzymes can be a demanding task, especially if the data are only partly available or come from various sources with unequal experimental setups"
