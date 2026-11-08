---
name: peptide-sequence-mismatch-alignment-msms
description: Use when evaluating de novo sequencing output against pseudo-ground-truth
  PSMs (e.g. MS-GF+/X!Tandem at 1% FDR) and you need the relative frequency of mismatch
  categories such as >6 wrongly assigned amino acids or last-3-AA inversions.
when_to_use_negative:
- When the goal is peptide identification itself rather than characterization of de
  novo prediction errors — use the database-search PSMs (MS-GF+/X!Tandem at 1% FDR)
  directly instead.
- When distinguishing isoleucine from leucine is required — MS/MS cannot discriminate
  these isomeric residues, so mismatch categories involving I/L cannot be resolved.
- When predictions come from tools not among the evaluated set (Novor, pNovo 3, DeepNovo,
  SMSNet, PointNovo, Casanovo) without re-deriving the 11 error categories for their
  output format.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0121
tools:
- name: pNovo 3
  role: Source of de novo peptide predictions whose mismatches are categorized
- name: SMSNet
  role: Source of de novo peptide predictions whose mismatches are categorized
- name: PointNovo
  role: Source of de novo peptide predictions whose mismatches are categorized
- name: Casanovo
  role: Source of de novo peptide predictions; its precursor mass filter explains
    its distinct error distribution
- name: MS-GF+
  role: Database search engine producing the pseudo-ground-truth PSMs at 1% FDR
- name: X!Tandem
  role: Database search engine producing the pseudo-ground-truth PSMs at 1% FDR
- name: SearchGUI
  role: Wraps MS-GF+/X!Tandem searches and PeptideShaker FDR filtering to build ground
    truth
- name: PeptideShaker
  role: Applies 1% FDR to SearchGUI results to yield pseudo-ground-truth PSMs
- name: Pyteomics
  role: Computes residue and prefix masses for the <0.1 Da / <0.5 Da matching criterion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:4bb1d7ec7c36bd78aa03f9b83540dc4307228ad4f7db4aaf645f19d79fc26f85
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# peptide-sequence-mismatch-alignment-msms

## Summary

Aligns de novo peptide predictions (pNovo 3, SMSNet, PointNovo, Casanovo) against pseudo-ground-truth PSMs to classify every incorrect prediction into one of 11 error-type categories, revealing tool-specific mismatch signatures on antibody MS/MS data.

## When to use

Use when evaluating de novo sequencing output against pseudo-ground-truth PSMs (e.g. MS-GF+/X!Tandem at 1% FDR) and you need the relative frequency of mismatch categories such as >6 wrongly assigned amino acids or last-3-AA inversions.

## When NOT to use

- When the goal is peptide identification itself rather than characterization of de novo prediction errors — use the database-search PSMs (MS-GF+/X!Tandem at 1% FDR) directly instead.
- When distinguishing isoleucine from leucine is required — MS/MS cannot discriminate these isomeric residues, so mismatch categories involving I/L cannot be resolved.
- When predictions come from tools not among the evaluated set (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) without re-deriving the 11 error categories for their output format.

## Inputs

- De novo prediction files for pNovo 3, SMSNet, PointNovo and Casanovo (from task_001)
- Pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker
- Antibody MS/MS data sets (three multi-enzymatic monoclonal antibody sets from PRIDE/MassIVE)

## Outputs

- error_type_frequencies.csv — table mirroring Table 3 with relative frequency of each of the 11 error categories per de novo tool across the three antibody data sets

## How to apply

Load the de novo prediction files for pNovo 3, SMSNet, PointNovo and Casanovo together with the pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker. Match predicted amino acids against ground-truth peptides using the paper's criterion: a residue counts as matched when its mass differs by <0.1 Da and the prefix masses before it differ by <0.5 Da (mass computations can be done with Pyteomics). Flag every non-identical prediction as incorrect and assign it to one of the 11 error categories (e.g. inversion of the last 3 amino acids, replacement of 1 AA by 1 or 2 AAs, more than six wrongly assigned amino acids). Compute the relative frequency of each error type per tool across the three antibody data sets and aggregate the counts into a table mirroring Table 3 (error_type_frequencies.csv).

## Related tools

- **pNovo 3** (Source of de novo peptide predictions whose mismatches are categorized)
- **SMSNet** (Source of de novo peptide predictions whose mismatches are categorized)
- **PointNovo** (Source of de novo peptide predictions whose mismatches are categorized)
- **Casanovo** (Source of de novo peptide predictions; its precursor mass filter explains its distinct error distribution)
- **MS-GF+** (Database search engine producing the pseudo-ground-truth PSMs at 1% FDR)
- **X!Tandem** (Database search engine producing the pseudo-ground-truth PSMs at 1% FDR)
- **SearchGUI** (Wraps MS-GF+/X!Tandem searches and PeptideShaker FDR filtering to build ground truth)
- **PeptideShaker** (Applies 1% FDR to SearchGUI results to yield pseudo-ground-truth PSMs)
- **Pyteomics** (Computes residue and prefix masses for the <0.1 Da / <0.5 Da matching criterion)

## Examples

```
matched = (abs(mass(pred_aa) - mass(true_aa)) < 0.1) and (abs(sum(mass(p) for p in pred[:i]) - sum(mass(t) for t in true[:i])) < 0.5)
```

## Evaluation signals

- pNovo 3, SMSNet and PointNovo show most errors in the more-than-six-wrongly-assigned-amino-acids category
- Casanovo shows a smaller fraction of large-mismatch errors, attributable to its precursor m/z filter
- Dominant sub-6-AA error types match the paper: last-3-AA inversions for PointNovo; 1-AA replaced by 1 or 2 AAs for SMSNet and pNovo 3
- Output table structure and categories mirror Table 3 (11 error categories, relative frequencies per tool per data set)
- Every non-identical prediction is assigned to exactly one of the 11 error categories (no unclassified incorrect predictions)

## Limitations

- Residues are matched only within tolerance (<0.1 Da residue mass, <0.5 Da prefix mass), so near-miss substitutions below tolerance may be counted as matches.
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D), blurring certain error categories.
- No version numbers are stated for any of the evaluated de novo sequencing tools, complicating exact reproduction.
- Casanovo's error distribution is shaped by its precursor mass filter, so category frequencies are not a pure measure of spectral interpretation quality.

## Evidence

- [methods] Match predicted amino acids against the ground-truth peptides using the paper's criterion: residues count as matched when their masses differ by <0.1 Da and the prefix masses before them differ by <0.5 Da.: [evidence span withheld — anchor status: not_found]
- [methods] Error categories cover large and small mismatches alike.: [evidence span withheld — anchor status: not_found]
- [results] Finding on the differing error distributions of the four tools.: [evidence span withheld — anchor status: not_found]
- [discussion] Rationale for Casanovo's distinct error profile.: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
- [intro] Study objective motivating the error-type investigation.: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
