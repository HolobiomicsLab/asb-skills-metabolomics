---
name: amino-acid-matching-metric-computation
description: Use when scoring de novo sequencing predictions (e.g. from Casanovo,
  PointNovo, DeepNovo, SMSNet, Novor, pNovo 3) against a PSM-derived ground truth
  to get peptide recall, amino acid recall/precision, or precision-recall AUC across
  enzymatic digests.
when_to_use_negative:
- The predictions will be assembled into full-length antibody sequences (e.g. with
  the de Bruijn assembler ALPS) rather than scored per peptide — assembly coverage
  is a different metric
- The goal is to resolve isomeric residues such as I/L, Q/AG or deamidated N/D — MS/MS-based
  amino-acid matching cannot discriminate these regardless of metric computation
- Scoring database-search PSMs themselves rather than de novo predictions; the 1%
  FDR filtering step already covers that
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0622
tools:
- name: Casanovo
  role: Deep-learning de novo sequencer whose per-spectrum predictions and confidence
    scores are matched against the pseudo-ground truth; highest AUC on four of six
    IgG1-Human-HC enzymatic data sets
- name: PointNovo
  role: Deep-learning de novo sequencer scored with the same amino-acid matching criteria;
    showed increased peptide recall versus spectrum-graph-based tools
- name: DeepNovo
  role: Deep-learning de novo sequencer whose predictions are evaluated with the amino-acid
    matching metric
- name: SMSNet
  role: Deep-learning de novo sequencer evaluated in the per-tool, per-enzyme metrics
    table
- name: Novor
  role: Spectrum-graph-based de novo sequencer run via DeNovoGUI; its predictions
    are scored with the same matching thresholds
- name: pNovo 3
  role: Spectrum-graph-based de novo sequencer with enzyme-specific pre-trained models;
    predictions scored with the same matching thresholds
- name: PeptideShaker
  role: Post-processes MS-GF+ and X!Tandem results to produce the 1% FDR pseudo-ground-truth
    PSMs against which de novo predictions are matched
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:73652476b3e5da147084dfd1cefdf339680416a520766c0cf2eae6edfda08b99
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
  - SMSNet
  - SearchGUI
  - X!Tandem
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# amino-acid-matching-metric-computation

## Summary

Computes amino-acid-level and peptide-level matching metrics (amino acid precision, amino acid recall, peptide recall, precision-recall AUC) for de novo peptide sequencing predictions against a pseudo-ground truth, using the mass-based residue matching criteria defined in the Beslic et al. antibody de novo sequencing benchmark.

## When to use

Use when scoring de novo sequencing predictions (e.g. from Casanovo, PointNovo, DeepNovo, SMSNet, Novor, pNovo 3) against a PSM-derived ground truth to get peptide recall, amino acid recall/precision, or precision-recall AUC across enzymatic digests.

## When NOT to use

- The predictions will be assembled into full-length antibody sequences (e.g. with the de Bruijn assembler ALPS) rather than scored per peptide — assembly coverage is a different metric
- The goal is to resolve isomeric residues such as I/L, Q/AG or deamidated N/D — MS/MS-based amino-acid matching cannot discriminate these regardless of metric computation
- Scoring database-search PSMs themselves rather than de novo predictions; the 1% FDR filtering step already covers that

## Inputs

- De novo sequencing predictions (peptide sequences with confidence scores) from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo or Casanovo
- Pseudo-ground-truth peptide-spectrum matches from SearchGUI/PeptideShaker (MS-GF+ and X!Tandem) filtered at 1% FDR with enzyme-specific cleavage parameters
- Per-enzyme digest groupings of the IgG1-Human-HC antibody data set (six enzymes, e.g. trypsin and lys-C)

## Outputs

- Per-tool, per-enzyme table of peptide recall, amino acid recall and amino acid precision (e.g. denovo_benchmark_metrics.csv)
- Precision-recall curves and AUC values per tool per enzymatic data set (Figure 1-style)

## How to apply

Match each predicted amino acid to the reference peptide when the individual residue mass difference is <0.1 Da and the prefix (cumulative) mass difference is <0.5 Da; a peptide counts as recalled when all its residues match under this rule. Run the comparison twice: first sweeping each tool's confidence-score threshold to draw precision-recall curves and compute AUC (as reported per enzyme in Figure 1), and second using all predictions regardless of confidence score to compute peptide recall, amino acid recall and amino acid precision per enzyme (Figure 2 A/B/C). Apply the same search parameters underlying the predictions — precursor tolerance 10 ppm, fragment mass tolerance 0.02 Da, fixed carbamidomethylation of cysteine (+57.02 Da), variable oxidation of methionine (+15.99 Da) and deamidation of N/Q (+0.98 Da) — when interpreting matched masses. Write the per-tool, per-enzyme metrics to a table such as denovo_benchmark_metrics.csv so patterns (e.g. Casanovo's highest AUC on four of six IgG1-Human-HC enzymatic data sets) can be validated against the reference results.

## Related tools

- **Casanovo** (Deep-learning de novo sequencer whose per-spectrum predictions and confidence scores are matched against the pseudo-ground truth; highest AUC on four of six IgG1-Human-HC enzymatic data sets)
- **PointNovo** (Deep-learning de novo sequencer scored with the same amino-acid matching criteria; showed increased peptide recall versus spectrum-graph-based tools)
- **DeepNovo** (Deep-learning de novo sequencer whose predictions are evaluated with the amino-acid matching metric)
- **SMSNet** (Deep-learning de novo sequencer evaluated in the per-tool, per-enzyme metrics table)
- **Novor** (Spectrum-graph-based de novo sequencer run via DeNovoGUI; its predictions are scored with the same matching thresholds)
- **pNovo 3** (Spectrum-graph-based de novo sequencer with enzyme-specific pre-trained models; predictions scored with the same matching thresholds)
- **PeptideShaker** (Post-processes MS-GF+ and X!Tandem results to produce the 1% FDR pseudo-ground-truth PSMs against which de novo predictions are matched)

## Examples

```
matched = [abs(m) < 0.1 and abs(p) < 0.5 for m, p in zip(residue_mass_diffs, prefix_mass_diffs)]
```

## Evaluation signals

- Reproduced metrics preserve the paper's pattern that Casanovo attains the highest AUC on four of the six IgG1-Human-HC enzymatic data sets and the highest peptide recall and amino acid precision
- All tools perform best on trypsin and lys-C digests relative to the other enzymes
- Residue matching is strictly enforced at mass difference <0.1 Da and prefix mass difference <0.5 Da — verify thresholds in code/config
- Metrics computed both with confidence-threshold sweeps (for PR AUC) and on all predictions regardless of confidence (for Figure 2 A/B/C metrics) — both output sets must exist
- Deep-learning tools (Casanovo, PointNovo) show higher peptide recall than spectrum-graph-based tools (Novor, pNovo 3) across enzymes

## Limitations

- Amino-acid matching cannot discriminate isomeric I/L; isoleucine/leucine discrimination requires MS3 fragmentation beyond MS/MS
- Q vs AG and deamidated N vs D cannot be resolved even with higher-resolution spectra and tools, so matches at these positions are mass-equivalent rather than sequence-identical
- Missing fragmentation sites, noisy spectra and long peptide sequences limit all de novo tools and thus depress the computed metrics
- Performance on non-tryptic enzymes varies with the cleavage pattern of the deep-learning tools' training data set
- The article provides no code or data repository URLs/accessions, so the metric computation must be reimplemented from the described thresholds

## Evidence

- [methods] counting amino acids as matched when their mass difference is <0.1 Da and the prefix mass difference is <0.5 Da: [evidence span withheld — anchor status: not_found]
- [methods] Sweep confidence-score thresholds per tool to draw precision-recall curves and compute the AUC across the six enzymes of IgG1-Human-HC.: [evidence span withheld — anchor status: not_found]
- [methods] Using all predictions regardless of confidence score, compute peptide recall, amino acid recall and amino acid precision per enzyme: [evidence span withheld — anchor status: not_found]
- [abstract] The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets: "The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets compared with spectrum-graph-based approaches."
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
