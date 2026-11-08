---
name: peptide-precision-recall-threshold-selection
description: Use when de novo peptide predictions from multiple sequencing tools (each
  reporting its own confidence score) must be filtered before assembly, and no absolute
  score cutoff is available because each tool calculates confidence differently.
when_to_use_negative:
- The predictions are already from a single tool with a published absolute score cutoff
  (e.g. a precursor mass filter), so per-tool calibration at 50% AA precision is unnecessary
- No database-search-based pseudo-ground truth (PeptideShaker 1% FDR) can be constructed
  for the data set, so precision cannot be computed
- The goal is post-assembly evaluation of sequence coverage rather than pre-assembly
  filtering of predictions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Novor
  role: de novo sequencing tool whose per-peptide confidence score is swept to find
    its 50% AA-precision threshold
- name: pNovo 3
  role: de novo sequencing tool whose per-peptide confidence score is swept to find
    its 50% AA-precision threshold
- name: DeepNovo
  role: de novo sequencing tool whose per-peptide confidence score is swept to find
    its 50% AA-precision threshold
- name: SMSNet
  role: de novo sequencing tool whose per-peptide confidence score is swept to find
    its 50% AA-precision threshold
- name: PointNovo
  role: de novo sequencing tool whose per-peptide confidence score is swept to find
    its 50% AA-precision threshold
- name: Casanovo
  role: de novo sequencing tool whose per-peptide confidence score is swept to find
    its 50% AA-precision threshold; its predictions then feed ALPS assembly
- name: PeptideShaker
  role: combines MS-GF+ and X!Tandem search results at 1% FDR to define the pseudo-ground-truth
    peptides used for precision computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:0aac28ae66982a0ac34547e391a1aed2bc98b5ecef58a858b2a853d5cd4b1dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - DeepNovo
  - Novor
  - PeptideShaker
  - PointNovo
  - SMSNet
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peptide-precision-recall-threshold-selection

## Summary

Selects a tool-specific confidence-score cutoff for de novo peptide sequencing predictions (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) at the point where amino-acid-level precision reaches 50% against a database-search pseudo-ground truth, so that only high-confidence predictions feed downstream de Bruijn assembly (ALPS).

## When to use

Use when de novo peptide predictions from multiple sequencing tools (each reporting its own confidence score) must be filtered before assembly, and no absolute score cutoff is available because each tool calculates confidence differently.

## When NOT to use

- The predictions are already from a single tool with a published absolute score cutoff (e.g. a precursor mass filter), so per-tool calibration at 50% AA precision is unnecessary
- No database-search-based pseudo-ground truth (PeptideShaker 1% FDR) can be constructed for the data set, so precision cannot be computed
- The goal is post-assembly evaluation of sequence coverage rather than pre-assembly filtering of predictions

## Inputs

- Per-tool de novo prediction sets (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo), each record carrying peptide sequence and tool-specific confidence score
- Pseudo-ground-truth peptide list from database search (MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR)

## Outputs

- filtered_predictions_per_tool.tsv — one filtered peptide set per tool, preserving peptide sequence, confidence score, and source tool
- Per-tool selected confidence-score threshold at which AA-level precision equals 50%

## How to apply

Load the per-tool de novo prediction sets and compute amino-acid-level precision at candidate confidence-score thresholds against a pseudo-ground truth built from database search (MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR). A predicted amino acid counts as matched when the mass difference is <0.1 Da and the prefix-mass difference is <0.5 Da. For each tool separately, choose the lowest confidence-score threshold at which AA-level precision reaches at least 50%, since each algorithm calculates its confidence score differently and thresholds are not transferable across tools. Remove peptide sequences below their tool's threshold and write the surviving peptides (sequence, confidence score, source tool) per tool to filtered_predictions_per_tool.tsv as the cleaned input for the ALPS de Bruijn assembly.

## Related tools

- **Novor** (de novo sequencing tool whose per-peptide confidence score is swept to find its 50% AA-precision threshold)
- **pNovo 3** (de novo sequencing tool whose per-peptide confidence score is swept to find its 50% AA-precision threshold)
- **DeepNovo** (de novo sequencing tool whose per-peptide confidence score is swept to find its 50% AA-precision threshold)
- **SMSNet** (de novo sequencing tool whose per-peptide confidence score is swept to find its 50% AA-precision threshold)
- **PointNovo** (de novo sequencing tool whose per-peptide confidence score is swept to find its 50% AA-precision threshold)
- **Casanovo** (de novo sequencing tool whose per-peptide confidence score is swept to find its 50% AA-precision threshold; its predictions then feed ALPS assembly)
- **PeptideShaker** (combines MS-GF+ and X!Tandem search results at 1% FDR to define the pseudo-ground-truth peptides used for precision computation)

## Evaluation signals

- For each tool, re-evaluating the retained peptide set against the PeptideShaker 1% FDR pseudo-ground truth yields amino-acid-level precision of at least 50% under the matched-amino-acid criterion (mass difference <0.1 Da, prefix-mass difference <0.5 Da)
- filtered_predictions_per_tool.tsv exists and contains exactly one filtered peptide set per evaluated tool
- The selected threshold is the lowest confidence-score cutoff at which each tool's AA precision is still >= 50%, so removal is minimal rather than over-aggressive
- Each threshold value is tool-specific; identical numeric cutoffs across tools signal the calibration was not done per tool

## Limitations

- Precision is measured against a pseudo-ground truth, not verified sequences, so the 50% AA-precision target inherits errors of the 1% FDR database search
- Isomeric/isobaric residues cannot be resolved by the matching criterion (I & L; Q & AG; deamidated N & D), affecting precision estimates
- Each algorithm calculates its confidence score differently, so thresholds derived per tool cannot be reused across tools or data sets

## Evidence

- [abstract] we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo): "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
- [discussion] While newly developed tools demonstrate the potential of de novo sequencing, advanced post-processing steps are necessary to improve their accuracy: "While newly developed tools demonstrate the potential of de novo sequencing, advanced post-processing steps are necessary to improve their accuracy"
- [discussion] The authors of Casanovo showed how a simple precursor mass filter yields much higher precision: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
