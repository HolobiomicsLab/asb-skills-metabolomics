---
name: search-engine-result-post-processing
description: Use when you need a trusted reference set of peptide identifications
  from antibody MS/MS spectra (MGF format) against a known protein/antibody FASTA,
  to benchmark de novo sequencing tools or validate PSMs.
when_to_use_negative:
- When no reference protein/antibody database exists and a truly database-independent
  de novo approach is required
- When the goal is de novo prediction itself (e.g. Casanovo, PointNovo) rather than
  producing a database-search ground truth for evaluation
- When spectra are not searchable against a sequence database (e.g. unmatched novel
  peptides absent from the reference FASTA)
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-GF+
  role: First database search engine executed via SearchGUI to identify PSMs against
    the antibody sequence database
- name: X!Tandem
  role: Second database search engine executed via SearchGUI; results combined with
    MS-GF+ to increase identification coverage
- name: SearchGUI
  role: Wraps and parameterizes both search engines (v.4.1.7), applying enzyme-specific
    cleavage and mass-tolerance settings
- name: PeptideShaker
  role: Post-processes and consolidates PSMs from both engines (v.2.2.2) prior to
    the 1% FDR filter
- name: Novor
  role: 'Downstream consumer: de novo predictions are evaluated against the FDR-filtered
    search results'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:eed9deed42e53c265d0e8f1bd6a2f52497b65f8ac4c84e01a06a07bb1ad371a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS-GF+
  - Novor
  - PeptideShaker
  - SearchGUI
  - X!Tandem
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# search-engine-result-post-processing

## Summary

Generate a high-confidence pseudo-ground-truth set of peptide-spectrum matches (PSMs) by combining results from two database search engines (MS-GF+ and X!Tandem) through SearchGUI and consolidating them in PeptideShaker, then filtering at 1% FDR. This reference set is used to evaluate de novo sequencing predictions on antibody MS/MS data.

## When to use

Use when you need a trusted reference set of peptide identifications from antibody MS/MS spectra (MGF format) against a known protein/antibody FASTA, to benchmark de novo sequencing tools or validate PSMs.

## When NOT to use

- When no reference protein/antibody database exists and a truly database-independent de novo approach is required
- When the goal is de novo prediction itself (e.g. Casanovo, PointNovo) rather than producing a database-search ground truth for evaluation
- When spectra are not searchable against a sequence database (e.g. unmatched novel peptides absent from the reference FASTA)

## Inputs

- MGF-converted MS/MS peak lists from antibody digest experiments
- Antibody reference sequences (protein database, FASTA) used as the search database
- Enzyme specification per input file (drives enzyme-specific cleavage parameters)

## Outputs

- PeptideShaker-processed, consolidated PSM results from MS-GF+ and X!Tandem
- Filtered pseudo-ground-truth PSM/peptide list at 1% FDR (e.g. 23,844 database-search-identified peptides across three antibody data sets)

## How to apply

Load the MGF-converted MS/MS experiments together with the antibody reference sequences, using those sequences as the protein database. Execute MS-GF+ and X!Tandem through SearchGUI (v.4.1.7) with cleavage parameters chosen per the enzyme of each input file, a precursor tolerance of 10 ppm, a fragment mass tolerance of 0.02 Da, carbamidomethylation of cysteine (C + 57.02 Da) as a fixed modification, and oxidation of methionine (M + 15.99 Da) plus deamidation of N and Q (+0.98 Da) as variable modifications. Post-process the combined search results with PeptideShaker (v.2.2.2) to consolidate peptide-spectrum matches from both engines. Finally, filter all resulting PSMs at a 1% false discovery rate to retain only high-confidence identifications as the ground-truth reference set.

## Related tools

- **MS-GF+** (First database search engine executed via SearchGUI to identify PSMs against the antibody sequence database)
- **X!Tandem** (Second database search engine executed via SearchGUI; results combined with MS-GF+ to increase identification coverage)
- **SearchGUI** (Wraps and parameterizes both search engines (v.4.1.7), applying enzyme-specific cleavage and mass-tolerance settings)
- **PeptideShaker** (Post-processes and consolidates PSMs from both engines (v.2.2.2) prior to the 1% FDR filter)
- **Novor** (Downstream consumer: de novo predictions are evaluated against the FDR-filtered search results)

## Evaluation signals

- The PeptideShaker output must contain PSMs filtered at 1% FDR (target-decoy controlled).
- The filtered peptide count should match the expected reference set (23,844 peptides across the three antibody data sets).
- Search parameters must be verifiable: 10 ppm precursor tolerance, 0.02 Da fragment tolerance, C + 57.02 Da fixed, M + 15.99 Da and N/Q + 0.98 Da variable.
- Each input file must have been searched with cleavage parameters matching its digestion enzyme.
- Both engines (MS-GF+ and X!Tandem) must have contributed to the consolidated PeptideShaker result, not just one.

## Limitations

- Database-search ground truth only covers peptides present in the reference antibody database; peptides absent from the FASTA cannot become pseudo-ground truth.
- The 1% FDR threshold still admits a small fraction (~1%) of false PSMs, which will propagate as label noise into de novo tool evaluation.
- Enzyme-specific cleavage parameters must be set correctly per input file; mismatched enzyme settings will bias the reference set.

## Evidence

- [methods] combined MS-GF+ and X!Tandem executed via SearchGUI and PeptideShaker: [evidence span withheld — anchor status: not_found]
- [methods] search parameter settings: [evidence span withheld — anchor status: not_found]
- [methods] FDR filter for ground truth: [evidence span withheld — anchor status: not_found]
- [discussion] rationale for filtering PSMs: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
