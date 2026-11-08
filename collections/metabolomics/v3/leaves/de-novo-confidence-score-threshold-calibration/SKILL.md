---
name: de-novo-confidence-score-threshold-calibration
description: Use when raw de novo peptide predictions from Novor, pNovo 3, DeepNovo,
  SMSNet, PointNovo or Casanovo must be filtered by their tool-specific confidence
  scores to reach a target amino-acid-level precision (here 50%) prior to ALPS de
  Bruijn assembly.
when_to_use_negative:
- The predictions are already filtered (e.g. a precursor mass filter has already been
  applied to raise precision), so re-calibrating thresholds would double-filter and
  lose recall.
- A de novo tool provides no per-prediction confidence score, making per-tool threshold
  sweeping impossible.
- The goal is to benchmark raw de novo tool performance (e.g. peptide recall comparison),
  since filtering to 50% AA precision would bias the comparison.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Novor
  role: Source of de novo peptide predictions with tool-specific confidence scores
    to be threshold-calibrated
- name: pNovo 3
  role: Source of de novo peptide predictions with tool-specific confidence scores
    to be threshold-calibrated
- name: DeepNovo
  role: Source of de novo peptide predictions with tool-specific confidence scores
    to be threshold-calibrated
- name: SMSNet
  role: Source of de novo peptide predictions with tool-specific confidence scores
    to be threshold-calibrated
- name: PointNovo
  role: Source of de novo peptide predictions with tool-specific confidence scores
    to be threshold-calibrated
- name: Casanovo
  role: Source of de novo peptide predictions with tool-specific confidence scores
    to be threshold-calibrated; its predictions were used for the final ALPS assembly
- name: PeptideShaker
  role: Combines MS-GF+ and X!Tandem database search results at 1% FDR into the pseudo-ground-truth
    peptides used as precision reference
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:3893b190433a1686bd937fd205ebdd3bb98c6c8205b1c19d417e82a1a55109f9
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

# de novo confidence-score threshold calibration

## Summary

Calibrates per-tool confidence-score thresholds so that retained de novo peptide predictions (from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) meet a target of 50% amino-acid-level precision before being fed into the ALPS de Bruijn assembly of antibody sequences.

## When to use

Use when raw de novo peptide predictions from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo or Casanovo must be filtered by their tool-specific confidence scores to reach a target amino-acid-level precision (here 50%) prior to ALPS de Bruijn assembly.

## When NOT to use

- The predictions are already filtered (e.g. a precursor mass filter has already been applied to raise precision), so re-calibrating thresholds would double-filter and lose recall.
- A de novo tool provides no per-prediction confidence score, making per-tool threshold sweeping impossible.
- The goal is to benchmark raw de novo tool performance (e.g. peptide recall comparison), since filtering to 50% AA precision would bias the comparison.

## Inputs

- Per-tool de novo peptide prediction sets from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo, each record with peptide sequence and tool-specific confidence score
- Pseudo-ground-truth peptide list from combined MS-GF+ and X!Tandem database search via PeptideShaker at 1% FDR

## Outputs

- filtered_predictions_per_tool.tsv with peptide sequence, confidence score and source tool, one filtered peptide set per de novo tool, serving as cleaned input for ALPS de Bruijn assembly

## How to apply

Load the per-tool de novo prediction sets (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo), each record carrying the peptide sequence and the tool-specific confidence score, plus the pseudo-ground-truth peptides obtained by database search with MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR. For each tool, sweep candidate confidence-score thresholds and, at each threshold, compute amino-acid-level precision against the pseudo-ground truth using the matched-amino-acid criterion (mass difference <0.1 Da and prefix-mass difference <0.5 Da). Because each algorithm calculates its confidence score differently, derive a tool-specific threshold: select the lowest confidence score at which AA-level precision is at least 50%. Remove peptide sequences below that per-tool threshold and write the surviving peptides to filtered_predictions_per_tool.tsv, preserving peptide sequence, confidence score, and source tool. This post-processing step is needed because the article notes that advanced post-processing steps are necessary to improve the accuracy of de novo sequencing tools before assembly with ALPS.

## Related tools

- **Novor** (Source of de novo peptide predictions with tool-specific confidence scores to be threshold-calibrated)
- **pNovo 3** (Source of de novo peptide predictions with tool-specific confidence scores to be threshold-calibrated)
- **DeepNovo** (Source of de novo peptide predictions with tool-specific confidence scores to be threshold-calibrated)
- **SMSNet** (Source of de novo peptide predictions with tool-specific confidence scores to be threshold-calibrated)
- **PointNovo** (Source of de novo peptide predictions with tool-specific confidence scores to be threshold-calibrated)
- **Casanovo** (Source of de novo peptide predictions with tool-specific confidence scores to be threshold-calibrated; its predictions were used for the final ALPS assembly)
- **PeptideShaker** (Combines MS-GF+ and X!Tandem database search results at 1% FDR into the pseudo-ground-truth peptides used as precision reference)

## Evaluation signals

- For each tool, re-evaluating the amino-acid-level precision of the retained peptide set against the PeptideShaker 1% FDR pseudo-ground truth yields at least 50%, mirroring the paper's filtering criterion.
- filtered_predictions_per_tool.tsv exists and contains exactly one filtered peptide set per de novo tool (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo).
- For every tool, each retained peptide's confidence score is at or above its selected tool-specific threshold, and peptides below the threshold are absent.
- The filtered sets are strictly smaller than the unfiltered per-tool prediction sets, confirming that threshold removal was applied.
- The matched-amino-acid evaluation used the stated criterion: mass difference <0.1 Da and prefix-mass difference <0.5 Da.

## Limitations

- Each algorithm calculates its confidence score differently, so the 50%-precision thresholds are tool-specific and cannot be transferred between tools.
- Precision is measured against a pseudo-ground truth (database search at 1% FDR), not true sequences, so calibration quality depends on the completeness of that reference.
- Discrimination of isomeric residues such as isoleucine and leucine cannot be achieved via MS/MS, so AA-level precision is insensitive to I/L errors.
- Low-quality spectra, missing fragmentation sites and long peptide sequences limit all de novo tools, so even calibrated, high-precision sets remain incomplete inputs for assembly.

## Evidence

- [abstract] we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo): "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
- [discussion] While newly developed tools demonstrate the potential of de novo sequencing, advanced post-processing steps are necessary to improve their accuracy: "While newly developed tools demonstrate the potential of de novo sequencing, advanced post-processing steps are necessary to improve their accuracy"
- [discussion] The authors of Casanovo showed how a simple precursor mass filter yields much higher precision: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
- [abstract] using the de Bruijn assembler ALPS and the predictions from Casanovo: "using the de Bruijn assembler ALPS and the predictions from Casanovo"
