---
name: precision-recall-curve-evaluation
description: Use when benchmarking de novo sequencing tools (Novor, pNovo 3, DeepNovo,
  SMSNet, PointNovo, Casanovo) against a peptide-spectrum-match ground truth and you
  need threshold-swept precision-recall AUCs to rank tools per enzyme.
when_to_use_negative:
- The goal is database-independent reconstruction of full-length antibody chains (de
  Bruijn assembly with ALPS), not per-spectrum sequencing quality benchmarking.
- Evaluating conventional database-search PSMs at fixed FDR rather than threshold-swept
  de novo predictions.
- The comparison concerns full antibody protein sequencing pipelines (e.g. Supernovo
  or Stitch workflows) rather than ranking individual sequencing algorithms.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: Novor
  role: Spectrum-graph-based sequencer evaluated under the threshold sweep (run via
    DeNovoGUI CLI)
- name: pNovo 3
  role: Sequencer run with pre-trained enzyme-specific models during the PR evaluation
- name: DeepNovo
  role: Deep-learning sequencer trained on the MassIVE HCD human proteome library
    and evaluated at swept confidence thresholds
- name: SMSNet
  role: Deep-learning sequencer trained and benchmarked alongside the other five tools
- name: PointNovo
  role: Deep-learning sequencer whose higher peptide recall is compared against spectrum-graph
    approaches in the PR analysis
- name: Casanovo
  role: Deep-learning sequencer expected to attain the highest precision-recall AUC
    across four of six enzymatic data sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:8519d1879f01d958cbf0cdf9fba8b8be2bc30f714ff67439f7d62ca349416531
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# precision-recall-curve-evaluation

## Summary

Sweeps per-tool confidence-score thresholds over de novo peptide sequencing predictions on multi-enzymatic antibody MS/MS data to draw precision-recall curves, compute AUCs, and report peptide recall, amino acid recall and amino acid precision per enzyme.

## When to use

Use when benchmarking de novo sequencing tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) against a peptide-spectrum-match ground truth and you need threshold-swept precision-recall AUCs to rank tools per enzyme.

## When NOT to use

- The goal is database-independent reconstruction of full-length antibody chains (de Bruijn assembly with ALPS), not per-spectrum sequencing quality benchmarking.
- Evaluating conventional database-search PSMs at fixed FDR rather than threshold-swept de novo predictions.
- The comparison concerns full antibody protein sequencing pipelines (e.g. Supernovo or Stitch workflows) rather than ranking individual sequencing algorithms.

## Inputs

- MGF-format MS/MS spectra files converted with ProteoWizard from PRIDE/MassIVE raw antibody data
- Pseudo-ground-truth peptide-spectrum matches from PeptideShaker filtered at 1% FDR with enzyme-specific cleavage parameters
- De novo sequencing predictions with confidence scores from Novor v.1.05 (via DeNovoGUI), pNovo 3 v.3.1.3, and DeepNovo/SMSNet/PointNovo/Casanovo trained on the MassIVE HCD human proteome library (1,114,503 peptides, 98:1:1 split, 10 epochs)

## Outputs

- Precision-recall curves per tool per enzyme with confidence thresholds swept
- Precision-recall AUC values per tool per enzyme (Figure 1 equivalents)
- denovo_benchmark_metrics.csv with peptide recall, amino acid recall and amino acid precision per tool per enzyme

## How to apply

Run all six sequencers on MGF-converted antibody spectra at a precursor tolerance of 10 ppm, fragment mass tolerance of 0.02 Da, fixed carbamidomethylation of cysteine (+57.02 Da) and variable oxidation of Met (+15.99 Da) and deamidation of N/Q (+0.98 Da). Build the pseudo-ground truth with MS-GF+ and X!Tandem via SearchGUI (v.4.1.7) and PeptideShaker (v.2.2.2) at 1% FDR with enzyme-specific cleavage parameters, then sweep each tool's confidence-score thresholds to draw precision-recall curves and compute AUC across the six enzymes of IgG1-Human-HC. Separately, using all predictions regardless of confidence, compute peptide recall, amino acid recall and amino acid precision per enzyme. Count amino acids as matched when their mass difference is <0.1 Da and the prefix mass difference is <0.5 Da. Write the per-tool, per-enzyme metrics table to denovo_benchmark_metrics.csv and check that the reproduced pattern matches the paper's finding (Casanovo highest AUC on four of six enzymatic data sets).

## Related tools

- **Novor** (Spectrum-graph-based sequencer evaluated under the threshold sweep (run via DeNovoGUI CLI))
- **pNovo 3** (Sequencer run with pre-trained enzyme-specific models during the PR evaluation)
- **DeepNovo** (Deep-learning sequencer trained on the MassIVE HCD human proteome library and evaluated at swept confidence thresholds)
- **SMSNet** (Deep-learning sequencer trained and benchmarked alongside the other five tools)
- **PointNovo** (Deep-learning sequencer whose higher peptide recall is compared against spectrum-graph approaches in the PR analysis)
- **Casanovo** (Deep-learning sequencer expected to attain the highest precision-recall AUC across four of six enzymatic data sets)

## Evaluation signals

- Casanovo attains the highest precision-recall AUC across four of the six enzymatic IgG1-Human-HC data sets; any reproduction should preserve this ordering.
- All tools perform best on trypsin and lys-C digests; deviating patterns suggest mis-set enzyme-specific cleavage parameters or tolerance mismatches.
- Amino acid matching uses mass difference <0.1 Da and prefix mass difference <0.5 Da; applying looser criteria inflates precision and recall relative to the reference table.
- The metrics table (Figure 1 AUCs and Figure 2 A/B/C values) is emitted as a per-tool, per-enzyme CSV (denovo_benchmark_metrics.csv).
- Deep-learning tools (Casanovo, PointNovo) should show increased peptide recall compared with spectrum-graph-based approaches across enzymes and data sets.

## Limitations

- DL tools applied to non-tryptic data vary in performance based on the cleavage pattern of the trained data set.
- Isoleucine/leucine, Q vs AG, and deamidated N vs D cannot be resolved even with higher-resolution spectra and tools, capping achievable precision.
- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools.
- Training enzyme-specific models is demanding when data are only partly available or come from sources with unequal experimental setups.

## Evidence

- [abstract] we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo): "we evaluated six recently developed de novo peptide sequencing algorithms (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo)"
- [abstract] The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets: "The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets compared with spectrum-graph-based approaches."
- [methods] Sweep confidence-score thresholds per tool to draw precision-recall curves and compute the AUC across the six enzymes of IgG1-Human-HC.: [evidence span withheld — anchor status: not_found]
- [methods] counting amino acids as matched when their mass difference is <0.1 Da and the prefix mass difference is <0.5 Da: [evidence span withheld — anchor status: not_found]
- [results] Casanovo achieved the highest precision-recall AUC across four of the six enzymatic data sets of IgG1-Human-HC: [evidence span withheld — anchor status: not_found]
