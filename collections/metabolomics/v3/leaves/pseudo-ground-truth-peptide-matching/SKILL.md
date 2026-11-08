---
name: pseudo-ground-truth-peptide-matching
description: Use when de novo sequencing predictions on antibody MS/MS data must be
  scored for accuracy or filtered against reference peptides from database search,
  e.g. to set confidence thresholds at 50% AA precision before ALPS assembly.
when_to_use_negative:
- When a true reference protein or antibody sequence is available — database-search-derived
  pseudo-ground truth is only needed when no a priori sequence knowledge exists
- When full-length reconstruction rather than prediction-quality assessment is the
  goal — that requires the de Bruijn assembler ALPS applied to the already-filtered
  predictions
- When distinguishing isobaric residues (e.g. I/L, Q vs AG, deamidated N vs D) — the
  matched-amino-acid criterion cannot resolve these via MS/MS
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: PeptideShaker
  role: Combines MS-GF+ and X!Tandem database-search identifications at 1% FDR to
    produce the pseudo-ground-truth peptide set
- name: MS-GF+
  role: Database search engine whose identifications form part of the pseudo-ground
    truth
- name: X!Tandem
  role: Database search engine whose identifications form part of the pseudo-ground
    truth
- name: Casanovo
  role: Deep-learning de novo predictor whose peptide sequences and confidence scores
    are matched against the pseudo-ground truth for threshold calibration
- name: PointNovo
  role: Deep-learning de novo predictor matched against the pseudo-ground truth with
    its own confidence-score threshold
- name: Novor
  role: Spectrum-graph-based de novo predictor matched against the pseudo-ground truth
    with its own confidence-score threshold
- name: pNovo 3
  role: Spectrum-graph-based de novo predictor matched against the pseudo-ground truth
    with its own confidence-score threshold
- name: DeepNovo
  role: Deep-learning de novo predictor matched against the pseudo-ground truth with
    its own confidence-score threshold
- name: SMSNet
  role: Deep-learning de novo predictor matched against the pseudo-ground truth with
    its own confidence-score threshold
- name: ALPS
  role: De Bruijn assembler consuming the threshold-filtered predictions produced
    via this matching step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:161181b3399937d4299b09268d6ae5e034d54619cce5d2b50c924f1446321daa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - Casanovo
  - DeepNovo
  - MS-GF+
  - Novor
  - PeptideShaker
  - PointNovo
  - SMSNet
  - X!Tandem
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# pseudo-ground-truth-peptide-matching

## Summary

Matches de novo peptide sequencing predictions (e.g. from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) against a pseudo-ground-truth peptide set derived from database search (MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR), using a matched-amino-acid criterion to compute amino-acid-level precision and calibrate per-tool confidence-score thresholds.

## When to use

Use when de novo sequencing predictions on antibody MS/MS data must be scored for accuracy or filtered against reference peptides from database search, e.g. to set confidence thresholds at 50% AA precision before ALPS assembly.

## When NOT to use

- When a true reference protein or antibody sequence is available — database-search-derived pseudo-ground truth is only needed when no a priori sequence knowledge exists
- When full-length reconstruction rather than prediction-quality assessment is the goal — that requires the de Bruijn assembler ALPS applied to the already-filtered predictions
- When distinguishing isobaric residues (e.g. I/L, Q vs AG, deamidated N vs D) — the matched-amino-acid criterion cannot resolve these via MS/MS

## Inputs

- Per-tool de novo prediction sets (peptide sequence + tool-specific confidence score) from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo
- Pseudo-ground-truth peptide set from database search (MS-GF+ and X!Tandem combined via PeptideShaker at 1% FDR)

## Outputs

- Amino-acid-level precision per tool per confidence-score threshold
- Per-tool confidence-score thresholds calibrated to 50% AA precision
- filtered_predictions_per_tool.tsv with peptide sequence, confidence score, and source tool for retained peptides

## How to apply

Build the pseudo-ground truth by combining MS-GF+ and X!Tandem identifications in PeptideShaker at 1% FDR, since no true reference exists for de novo antibody sequencing. For each de novo prediction, apply the matched-amino-acid criterion: an amino acid counts as correct when its mass difference is <0.1 Da and its prefix-mass difference is <0.5 Da against the reference peptide. Sweep candidate confidence-score thresholds per tool and compute amino-acid-level precision at each; because each algorithm calculates its confidence score differently (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo), thresholds must be derived independently per tool. Select the lowest threshold at which AA precision is at least 50%, remove predictions scoring below it, and use the retained peptides as cleaned input for downstream de Bruijn assembly with ALPS.

## Related tools

- **PeptideShaker** (Combines MS-GF+ and X!Tandem database-search identifications at 1% FDR to produce the pseudo-ground-truth peptide set)
- **MS-GF+** (Database search engine whose identifications form part of the pseudo-ground truth)
- **X!Tandem** (Database search engine whose identifications form part of the pseudo-ground truth)
- **Casanovo** (Deep-learning de novo predictor whose peptide sequences and confidence scores are matched against the pseudo-ground truth for threshold calibration)
- **PointNovo** (Deep-learning de novo predictor matched against the pseudo-ground truth with its own confidence-score threshold)
- **Novor** (Spectrum-graph-based de novo predictor matched against the pseudo-ground truth with its own confidence-score threshold)
- **pNovo 3** (Spectrum-graph-based de novo predictor matched against the pseudo-ground truth with its own confidence-score threshold)
- **DeepNovo** (Deep-learning de novo predictor matched against the pseudo-ground truth with its own confidence-score threshold)
- **SMSNet** (Deep-learning de novo predictor matched against the pseudo-ground truth with its own confidence-score threshold)
- **ALPS** (De Bruijn assembler consuming the threshold-filtered predictions produced via this matching step)

## Evaluation signals

- For each tool, re-evaluate the retained peptide set against the pseudo-ground truth and confirm amino-acid-level precision is at least 50%
- Confirm filtered_predictions_per_tool.tsv exists and contains exactly one filtered peptide set per tool (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo)
- Verify each retained peptide record preserves peptide sequence, confidence score, and source tool
- Check the matched-amino-acid criterion was applied with mass difference <0.1 Da and prefix-mass difference <0.5 Da
- Confirm thresholds are per-tool (scores are not comparable across algorithms) rather than a single shared cutoff

## Limitations

- The pseudo-ground truth is database-search-derived, so matching quality depends on the 1% FDR identification set and cannot validate peptides absent from database search
- Each algorithm calculates its confidence score differently, so thresholds are not transferable between tools
- Particular amino acids (I & L; Q & AG; deamidated N & D) cannot be resolved even with higher-resolution spectra, capping achievable precision
- The matched-amino-acid criterion's 0.1 Da / 0.5 Da tolerances may mis-handle mass segment errors that later limit ALPS assembly coverage, especially on heavy chains

## Evidence

- [methods] threshold calibration at 50% AA precision: [evidence span withheld — anchor status: not_found]
- [methods] pseudo-ground truth source: [evidence span withheld — anchor status: not_found]
- [methods] matching criterion: [evidence span withheld — anchor status: not_found]
- [abstract] downstream assembly use: "using the de Bruijn assembler ALPS and the predictions from Casanovo"
- [intro] evaluated tools: "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
