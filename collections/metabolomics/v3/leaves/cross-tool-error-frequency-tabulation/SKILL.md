---
name: cross-tool-error-frequency-tabulation
description: Use when comparing de novo peptide sequencing predictions (e.g. pNovo
  3, SMSNet, PointNovo, Casanovo) against pseudo-ground-truth PSMs and you need to
  characterize which error types dominate each tool's incorrect predictions.
when_to_use_negative:
- When you only need peptide recall/precision metrics for each tool rather than a
  breakdown of why incorrect predictions failed
- When the input spectra lack a database-search-derived pseudo-ground truth (MS-GF+/X!Tandem
  PSMs at 1% FDR), since error categories cannot be assigned without reference peptides
- When evaluating protein/antibody assembly quality (e.g. ALPS sequence coverage)
  rather than per-peptide prediction errors
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pNovo 3
  role: de novo sequencer whose incorrect predictions are categorized by error type
- name: SMSNet
  role: de novo sequencer whose incorrect predictions are categorized by error type
- name: PointNovo
  role: de novo sequencer whose incorrect predictions are categorized by error type
- name: Casanovo
  role: deep learning de novo sequencer; its precursor mass filter shapes its error
    distribution
- name: MS-GF+
  role: database search engine providing pseudo-ground-truth PSMs for error categorization
- name: X!Tandem
  role: database search engine providing pseudo-ground-truth PSMs for error categorization
- name: SearchGUI
  role: runs the database searches used to build the pseudo-ground truth
- name: PeptideShaker
  role: PSM validation at 1% FDR to define the pseudo-ground truth
- name: Pyteomics
  role: Python library for parsing and matching predicted vs. ground-truth peptide
    sequences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:9355f422160b178c1b07247e78c8ab22086e95d459204868b260753b8bdd00ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - MS-GF+
  - PeptideShaker
  - PointNovo
  - Pyteomics
  - SMSNet
  - SearchGUI
  - X!Tandem
  - pNovo 3
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-tool-error-frequency-tabulation

## Summary

Tabulates the relative frequencies of error types among incorrect de novo peptide sequence predictions across multiple de novo sequencing tools, using a mass-based residue-matching criterion to classify mismatches into defined error categories (e.g. inversions, substitutions, >6 wrongly assigned amino acids).

## When to use

Use when comparing de novo peptide sequencing predictions (e.g. pNovo 3, SMSNet, PointNovo, Casanovo) against pseudo-ground-truth PSMs and you need to characterize which error types dominate each tool's incorrect predictions.

## When NOT to use

- When you only need peptide recall/precision metrics for each tool rather than a breakdown of why incorrect predictions failed
- When the input spectra lack a database-search-derived pseudo-ground truth (MS-GF+/X!Tandem PSMs at 1% FDR), since error categories cannot be assigned without reference peptides
- When evaluating protein/antibody assembly quality (e.g. ALPS sequence coverage) rather than per-peptide prediction errors

## Inputs

- De novo peptide prediction files from pNovo 3, SMSNet, PointNovo and Casanovo (from task_001)
- Pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker
- MS/MS spectra from three multi-enzymatic monoclonal antibody data sets (downloaded from PRIDE and MassIVE)

## Outputs

- error_type_frequencies.csv — relative frequency of each of the 11 error categories per tool, mirroring Table 3
- Per-tool error-type distribution enabling comparison of dominant error classes across de novo sequencers

## How to apply

Load the de novo prediction files for each tool (pNovo 3, SMSNet, PointNovo, Casanovo) together with pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker. Match predicted amino acids against ground-truth peptides using the paper's criterion: residues count as matched when their masses differ by <0.1 Da and the prefix masses before them differ by <0.5 Da. Flag every non-identical prediction as incorrect and assign it to one of 11 error categories (e.g. inversion of the last 3 amino acids, replacement of 1 AA by 1 or 2 AAs, more than six wrongly assigned amino acids). Compute the relative frequency of each error type per tool across the antibody data sets and aggregate into a table mirroring Table 3. Interpret differences mechanistically: Casanovo's lower fraction of large-mismatch errors is attributable to its precursor m/z filter, whereas spectrum-graph tools show most errors in the >6 wrongly assigned AA category.

## Related tools

- **pNovo 3** (de novo sequencer whose incorrect predictions are categorized by error type)
- **SMSNet** (de novo sequencer whose incorrect predictions are categorized by error type)
- **PointNovo** (de novo sequencer whose incorrect predictions are categorized by error type)
- **Casanovo** (deep learning de novo sequencer; its precursor mass filter shapes its error distribution)
- **MS-GF+** (database search engine providing pseudo-ground-truth PSMs for error categorization)
- **X!Tandem** (database search engine providing pseudo-ground-truth PSMs for error categorization)
- **SearchGUI** (runs the database searches used to build the pseudo-ground truth)
- **PeptideShaker** (PSM validation at 1% FDR to define the pseudo-ground truth)
- **Pyteomics** (Python library for parsing and matching predicted vs. ground-truth peptide sequences)

## Evaluation signals

- pNovo 3, SMSNet and PointNovo show most errors in the 'more than six wrongly assigned amino acids' category
- Casanovo shows a smaller fraction of large-mismatch (>6 AA) errors, attributable to its precursor m/z filter
- Dominant sub-6-AA error types match the paper: last-3-AA inversions for PointNovo; 1-AA replaced by 1 or 2 AAs for SMSNet and pNovo 3
- All incorrect predictions are assigned to exactly one of the 11 error categories with no unclassified residue-level mismatches
- The aggregated table reproduces the structure of Table 3 with relative frequencies per tool across all three antibody data sets

## Limitations

- Error classification depends on the matching criterion (mass difference <0.1 Da per residue, prefix mass difference <0.5 Da); isomeric residues such as isoleucine/leucine cannot be discriminated via MS/MS and require MS3 fragmentation
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D), which can confound error categorization
- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools, shaping the error distribution itself

## Evidence

- [discussion] The authors of Casanovo showed how a simple precursor mass filter yields much higher precision: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
- [intro] Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions.: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [discussion] DL-based algorithms predict a higher amount of correct peptide sequences compared with conventional spectrum-graph-based methods: "DL-based algorithms predict a higher amount of correct peptide sequences compared with conventional spectrum-graph-based methods"
