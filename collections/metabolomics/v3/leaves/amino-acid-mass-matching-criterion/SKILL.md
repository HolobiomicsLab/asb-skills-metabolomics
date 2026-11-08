---
name: amino-acid-mass-matching-criterion
description: Use when comparing de novo peptide predictions to reference peptide sequences
  where exact string equality is too strict, e.g. to classify incorrect predictions
  into error categories (last-3-AA inversions, 1-AA replacements, >6 wrongly assigned
  AAs) across tools like Casanovo or PointNovo.
when_to_use_negative:
- When the goal is exact peptide identification against a database (use the 1% FDR
  MS-GF+/X!Tandem SearchGUI/PeptideShaker results directly instead of tolerance-based
  matching)
- When assessing full antibody protein assembly quality (e.g. ALPS sequence coverage
  on light/heavy chains) rather than per-peptide error classification
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: pNovo 3
  role: de novo prediction source whose errors are classified by the matching criterion
- name: SMSNet
  role: de novo prediction source whose errors are classified by the matching criterion
- name: PointNovo
  role: de novo prediction source whose errors are classified by the matching criterion
- name: Casanovo
  role: de novo prediction source whose error distribution is compared; its precursor
    mass filter explains fewer large-mismatch errors
- name: MS-GF+
  role: database search engine producing the 1% FDR pseudo-ground-truth PSMs
- name: X!Tandem
  role: database search engine producing the 1% FDR pseudo-ground-truth PSMs
- name: SearchGUI
  role: wraps the MS-GF+/X!Tandem searches used to build the ground truth
- name: PeptideShaker
  role: PSM validation at 1% FDR for the ground-truth peptides
- name: Pyteomics
  role: computes residue and prefix peptide masses for the <0.1 Da / <0.5 Da tolerance
    checks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:ef80da7252cb4f035d99c112b420acffd72cf87284d656ab7dcaa55f93f31e24
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# amino-acid-mass-matching-criterion

## Summary

Applies the paper's tolerance-based residue matching rule to score de novo peptide predictions against pseudo-ground-truth PSMs: a predicted amino acid counts as matched when its mass differs by <0.1 Da and the prefix mass before it differs by <0.5 Da. This is the scoring basis for the error-type frequency analysis comparing pNovo 3, SMSNet, PointNovo and Casanovo on antibody MS/MS data.

## When to use

Use when comparing de novo peptide predictions to reference peptide sequences where exact string equality is too strict, e.g. to classify incorrect predictions into error categories (last-3-AA inversions, 1-AA replacements, >6 wrongly assigned AAs) across tools like Casanovo or PointNovo.

## When NOT to use

- When the goal is exact peptide identification against a database (use the 1% FDR MS-GF+/X!Tandem SearchGUI/PeptideShaker results directly instead of tolerance-based matching)
- When assessing full antibody protein assembly quality (e.g. ALPS sequence coverage on light/heavy chains) rather than per-peptide error classification

## Inputs

- De novo peptide prediction files from pNovo 3, SMSNet, PointNovo and Casanovo (output of task_001)
- Pseudo-ground-truth PSMs from MS-GF+ and X!Tandem searches at 1% FDR (SearchGUI/PeptideShaker) on the three multi-enzymatic antibody data sets

## Outputs

- Per-prediction match/mismatch labels under the <0.1 Da residue / <0.5 Da prefix mass criterion
- Relative frequency of each of the 11 error types per tool (Table 3 mirror), written to error_type_frequencies.csv

## How to apply

Load de novo prediction files (pNovo 3, SMSNet, PointNovo, Casanovo) alongside pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker. Align each predicted peptide against its ground-truth peptide position by position, using Pyteomics to compute residue and prefix masses. Flag a residue as matched only under the dual tolerance criterion: residue mass difference <0.1 Da AND prefix mass difference <0.5 Da; all non-identical predictions are flagged incorrect. Assign each incorrect prediction to one of the paper's 11 error categories (e.g. inversion of the last 3 amino acids, replacement of 1 AA by 1 or 2 AAs, more than six wrongly assigned amino acids), compute relative frequencies per tool across the three antibody data sets, and write the Table-3-style table to error_type_frequencies.csv. The dual tolerance matters because a single mass error shifts all downstream residues, so the prefix-mass check distinguishes a localized substitution from a propagated mass offset.

## Related tools

- **pNovo 3** (de novo prediction source whose errors are classified by the matching criterion)
- **SMSNet** (de novo prediction source whose errors are classified by the matching criterion)
- **PointNovo** (de novo prediction source whose errors are classified by the matching criterion)
- **Casanovo** (de novo prediction source whose error distribution is compared; its precursor mass filter explains fewer large-mismatch errors)
- **MS-GF+** (database search engine producing the 1% FDR pseudo-ground-truth PSMs)
- **X!Tandem** (database search engine producing the 1% FDR pseudo-ground-truth PSMs)
- **SearchGUI** (wraps the MS-GF+/X!Tandem searches used to build the ground truth)
- **PeptideShaker** (PSM validation at 1% FDR for the ground-truth peptides)
- **Pyteomics** (computes residue and prefix peptide masses for the <0.1 Da / <0.5 Da tolerance checks)

## Examples

```
from pyteomics import mass; matched = abs(aa_mass_pred - aa_mass_true) < 0.1 and abs(prefix_mass_pred - prefix_mass_true) < 0.5
```

## Evaluation signals

- pNovo 3, SMSNet and PointNovo show most incorrect predictions in the more-than-six-wrongly-assigned-amino-acids category
- Casanovo shows a smaller fraction of large-mismatch (>6 wrongly assigned AAs) errors, attributable to its precursor m/z filter
- Dominant sub-6-AA error types reproduce the paper: last-3-AA inversions for PointNovo, 1-AA replaced by 1 or 2 AAs for SMSNet and pNovo 3
- All identical string matches are also accepted by the mass criterion (no false mismatches among exact matches)
- Output error_type_frequencies.csv schema matches Table 3 (11 error categories × 4 tools, relative frequencies summing to 1 per tool)

## Limitations

- The criterion cannot discriminate isomeric residues such as isoleucine/leucine (identical masses); I/L, Q/AG and deamidated N/D confusions are unresolvable even at these tolerances
- Matching depends on the quality of the pseudo-ground truth at 1% FDR, so reference peptides themselves may contain errors
- Casanovo's precursor mass filter suppresses some error types, making cross-tool error distributions not directly comparable in absolute terms

## Evidence

- [methods] residues count as matched when their masses differ by <0.1 Da and the prefix masses before them differ by <0.5 Da: [evidence span withheld — anchor status: not_found]
- [methods] Flag every non-identical prediction as incorrect and assign it to one of the 11 error categories: [evidence span withheld — anchor status: not_found]
- [discussion] The authors of Casanovo showed how a simple precursor mass filter yields much higher precision: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
- [intro] Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions.: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
