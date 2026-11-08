---
name: pseudo-ground-truth-comparison-database-search
description: Use when evaluating de novo peptide sequencing tools on antibody MS/MS
  data and no authoritative reference sequences exist — a database search against
  the known antibody sequences provides the pseudo-ground-truth PSMs needed to compute
  precision, recall and error-type distributions.
when_to_use_negative:
- When a true reference antibody sequence is unavailable or the protein database is
  too incomplete for a meaningful database search — the pseudo-ground truth would
  then be unreliable
- When the goal is to discriminate isomeric residues such as isoleucine/leucine, which
  the article notes cannot be resolved via MS/MS and require MS3 fragmentation
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS-GF+
  role: Database search engine used (with X!Tandem) to generate candidate PSMs against
    the antibody protein database
- name: X!Tandem
  role: Second database search engine combined with MS-GF+ to build the pseudo-ground-truth
    PSM set
- name: SearchGUI
  role: Wrapper (v.4.1.7) running MS-GF+ and X!Tandem with enzyme-specific cleavage
    parameters
- name: PeptideShaker
  role: Post-processing (v.2.2.2) of SearchGUI results, filtering PSMs at 1% FDR to
    yield the pseudo-ground truth
- name: ProteoWizard
  role: Converts raw MS/MS files from PRIDE/MassIVE to Mascot Generic Format (MGF)
    prior to searching
- name: Pyteomics
  role: Used to match predicted amino acids against ground-truth peptides and classify
    errors
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:81df85e4ac8ce062fd80d6018f12035a5c1e1ae66c34e0c4a400504ab9cb54ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - DeNovoGUI
  - DeepNovo
  - MS-GF+
  - Novor
  - PeptideShaker
  - PointNovo
  - ProteoWizard
  - Pyteomics
  - SMSNet
  - SearchGUI
  - X!Tandem
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# Pseudo-ground-truth comparison database search

## Summary

Builds a pseudo-ground-truth set of peptide-spectrum matches (PSMs) for antibody MS/MS data by database searching with MS-GF+ and X!Tandem via SearchGUI (v.4.1.7) and PeptideShaker (v.2.2.2) at 1% FDR, so that de novo sequencing predictions (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) can be scored against an independent reference.

## When to use

Use when evaluating de novo peptide sequencing tools on antibody MS/MS data and no authoritative reference sequences exist — a database search against the known antibody sequences provides the pseudo-ground-truth PSMs needed to compute precision, recall and error-type distributions.

## When NOT to use

- When a true reference antibody sequence is unavailable or the protein database is too incomplete for a meaningful database search — the pseudo-ground truth would then be unreliable
- When the goal is to discriminate isomeric residues such as isoleucine/leucine, which the article notes cannot be resolved via MS/MS and require MS3 fragmentation

## Inputs

- Raw antibody MS/MS files from PRIDE/MassIVE (converted to Mascot Generic Format, MGF, via ProteoWizard)
- Antibody protein sequence database (e.g. IgG1-Human-HC sequences) as FASTA
- De novo prediction files from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and/or Casanovo
- Search settings: precursor tolerance 10 ppm, fragment mass tolerance 0.02 Da, fixed carbamidomethylation of cysteine (+57.02 Da), variable oxidation of methionine (+15.99 Da) and deamidation of N/Q (+0.98 Da)

## Outputs

- Pseudo-ground-truth PSM list filtered at 1% FDR (from PeptideShaker)
- Per-tool, per-enzyme benchmark metrics table (peptide recall, amino acid recall, amino acid precision, precision-recall AUC) written to denovo_benchmark_metrics.csv
- Error-type frequency table (11 categories, e.g. inversion of the last 3 amino acids, more than six wrongly assigned amino acids) written to error_type_frequencies.csv

## How to apply

Convert the raw antibody MS/MS files (downloaded from PRIDE/MassIVE) to Mascot Generic Format (MGF) with ProteoWizard, then run MS-GF+ and X!Tandem through SearchGUI (v.4.1.7) using the antibody sequences (e.g. IgG1-Human-HC) as the protein database with enzyme-specific cleavage parameters. Post-process the searches in PeptideShaker (v.2.2.2) and filter peptide-spectrum matches at 1% FDR to obtain the pseudo-ground truth. Match each de novo tool's predicted amino acids against these ground-truth peptides using the paper's criterion: residues count as matched when their masses differ by <0.1 Da and the prefix masses before them differ by <0.5 Da. The rationale is that an independent, FDR-controlled database search provides a consistent reference across tools and enzymes (trypsin, lys-C, etc.) without which precision/recall and error-type comparisons (e.g. the 11 error categories in Table 3) cannot be computed.

## Related tools

- **MS-GF+** (Database search engine used (with X!Tandem) to generate candidate PSMs against the antibody protein database)
- **X!Tandem** (Second database search engine combined with MS-GF+ to build the pseudo-ground-truth PSM set)
- **SearchGUI** (Wrapper (v.4.1.7) running MS-GF+ and X!Tandem with enzyme-specific cleavage parameters)
- **PeptideShaker** (Post-processing (v.2.2.2) of SearchGUI results, filtering PSMs at 1% FDR to yield the pseudo-ground truth)
- **ProteoWizard** (Converts raw MS/MS files from PRIDE/MassIVE to Mascot Generic Format (MGF) prior to searching)
- **Pyteomics** (Used to match predicted amino acids against ground-truth peptides and classify errors)

## Evaluation signals

- All retained PSMs pass a 1% FDR filter computed by PeptideShaker over the combined MS-GF+ and X!Tandem searches
- Search parameters match the paper: 10 ppm precursor tolerance, 0.02 Da fragment tolerance, fixed +57.02 Da carbamidomethyl-Cys, variable +15.99 Da Met oxidation and +0.98 Da N/Q deamidation
- The derived benchmark metrics preserve the paper's pattern: Casanovo attains the highest precision-recall AUC across four enzymatic IgG1-Human-HC data sets and the highest amino acid precision, with all tools performing best on trypsin and lys-C
- Error-type analysis reproduces Table 3: pNovo 3, SMSNet and PointNovo show most errors in the more-than-six-wrongly-assigned-AA category; dominant sub-6-AA errors are last-3-AA inversions for PointNovo and 1-AA-replaced-by-1-or-2-AAs for SMSNet and pNovo 3
- Amino acid matching uses the paper's threshold: mass difference <0.1 Da and prefix mass difference <0.5 Da

## Limitations

- The pseudo-ground truth is only as good as the antibody protein database; heavy chains showed low sequence coverage and accuracy, demonstrating complete de novo protein sequencing remains challenging
- No version numbers are stated for all evaluated de novo tools and no code/data repository URLs or accessions are provided, limiting reproducibility
- Q vs AG and deamidated N vs D cannot be resolved even with higher-resolution spectra, so ground-truth comparisons conflate these mass-degenerate assignments

## Evidence

- [methods] Build the pseudo-ground truth by running MS-GF+ and X!Tandem through SearchGUI (v.4.1.7) and post-processing in PeptideShaker (v.2.2.2) against the antibody sequences as the protein database: [evidence span withheld — anchor status: not_found]
- [methods] filtering peptide-spectrum matches at 1% FDR with enzyme-specific cleavage parameters: [evidence span withheld — anchor status: not_found]
- [methods] counting amino acids as matched when their mass difference is <0.1 Da and the prefix mass difference is <0.5 Da: [evidence span withheld — anchor status: not_found]
- [intro] The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36].: "The data sets were downloaded from the proteomics MS data repositories PRIDE [35] and MassIVE [36]."
