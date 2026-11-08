---
name: psm-target-decoy-classification
description: Use when unfiltered database-search PSMs need post-processing to improve
  identification sensitivity — e.g., immunopeptidomics or metaproteomics where features
  like MS²PIP fragment-intensity or DeepLC retention-time predictions must be folded
  into a final score.
when_to_use_negative:
- When the PSM list is already fully rescoring-feature-augmented and re-scored with
  a converged classifier — re-running risks overfitting the training FDR
- When PSMs lack target/decoy labels or a decoy pattern is undefined; the classifier
  cannot distinguish positives from negatives
- For top-down proteoform identification, where the ProForma peptidoform representation
  and PSM feature tables may not apply
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ristretto
  role: Lean reimplementation of the Percolator semi-supervised rescoring algorithm;
    trains the linear SVM or LDA classifier and reports scores, q-values, PEPs
  repo: https://github.com/CompOmics/ristretto
- name: MS2Rescore
  role: Orchestrates feature generation (MS2PIP, DeepLC) and invokes ristretto/Percolator
    for the target-decoy classification step
  repo: https://github.com/compomics/ms2rescore
- name: Mumble
  role: Generates candidate modified PSMs for open-modification mass-shift proposals,
    expanding the PSM pool before rescoring
  repo: https://github.com/compomics/mumble
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:5fbf5ea5a73bbcdb9bc44984d2384a437e92ea4b46b0526c96901c267346854a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/CompOmics/ristretto
  tools:
  - MS2Rescore
  - Mumble
  - Percolator
  - Ristretto
  - ristretto
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# psm-target-decoy-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a semi-supervised classifier (linear SVM à la Percolator/ristretto) to separate correct target PSMs from incorrect decoy PSMs, rescoring peptide-spectrum matches and yielding calibrated q-values and PEPs.

## When to use

Use when unfiltered database-search PSMs need post-processing to improve identification sensitivity — e.g., immunopeptidomics or metaproteomics where features like MS²PIP fragment-intensity or DeepLC retention-time predictions must be folded into a final score.

## When NOT to use

- When the PSM list is already fully rescoring-feature-augmented and re-scored with a converged classifier — re-running risks overfitting the training FDR
- When PSMs lack target/decoy labels or a decoy pattern is undefined; the classifier cannot distinguish positives from negatives
- For top-down proteoform identification, where the ProForma peptidoform representation and PSM feature tables may not apply

## Inputs

- Unfiltered PSM feature table from a database search (mzid, TSV, MaxQuant, PEAKS, X!Tandem, etc.) with target/decoy labels and spectrum identifiers
- Spectrum files (mzML / MGF) linked to PSMs for fragment-annotation feature generation
- Optional: Mumble-generated candidate PSMs for mass-shift modification proposals
- Proteome FASTA (when generating amino-acid-combination decoys)

## Outputs

- Rescored PSM table with per-PSM scores, q-values, and posterior error probabilities (ms2rescore output rescored_psms.ms2rescore.tsv)
- Peptide- and protein-level q-value/PEP summaries from the trained classifier
- Trained classifier model object reusable for retraining or transfer

## How to apply

Parse search-engine results into a PSM feature table, mapping search-engine modification labels to HUPO-PSI ProForma v2 and linking each PSM to its spectrum via spectrum_id_pattern / psm_id_pattern regexes. Split PSMs into positives (top-ranked target PSMs at the training FDR, default train_fdr=0.01) and negatives (decoy PSMs identified by id_decoy_pattern). Train a linear SVM (ristretto's 'svm' model, matching Percolator/mokapot's Käll et al. 2007 semi-supervised iterative procedure) or the faster single-pass 'lda'; tune hyperparameters once via cross-validation rather than every iteration. Apply the trained model to all PSMs, then filter lower-ranking PSMs (max_psm_rank_output, default 1) before final FDR calculation so that output q-values remain valid. Output per-PSM rescored scores, q-values, and PEPs at the PSM, peptide, and protein level.

## Related tools

- **ristretto** (Lean reimplementation of the Percolator semi-supervised rescoring algorithm; trains the linear SVM or LDA classifier and reports scores, q-values, PEPs) — https://github.com/CompOmics/ristretto
- **MS2Rescore** (Orchestrates feature generation (MS2PIP, DeepLC) and invokes ristretto/Percolator for the target-decoy classification step) — https://github.com/compomics/ms2rescore
- **Mumble** (Generates candidate modified PSMs for open-modification mass-shift proposals, expanding the PSM pool before rescoring) — https://github.com/compomics/mumble

## Examples

```
ms2rescore -i input_psms.tsv --fasta-file proteins.fasta --ristretto-train-fdr 0.01 --ristretto-model svm
```

## Evaluation signals

- Every input PSM receives a finite rescored score; no NaNs in output columns
- All output q-values lie within [0, 1] and decrease monotonically as score thresholds loosen
- Decoy-derived FDR at the chosen train_fdr threshold (0.01 by default) is respected in the final PSM set
- Lower-ranking PSMs filtered out before final FDR calculation, matching max_psm_rank_output configuration
- Rescoring increases the number of peptide IDs at the same FDR threshold relative to the unrescored input

## Limitations

- Hyperparameters are tuned once rather than every iteration, trading some accuracy for speed
- The 'lda' model is faster but less powerful than the default 'svm'
- Mumble is beta software; unusual input files or uncommon modifications may produce errors that require manual review
- Search engines like MaxQuant may not report fixed modifications, so users must supply them to correctly rescore PSMs

## Evidence

- [readme] ristretto trains a classifier to separate correct from incorrect matches and reports q-values/PEPs: [evidence span withheld — anchor status: external_doc]
- [readme] The SVM model follows the semi-supervised iterative procedure of Käll et al. 2007: [evidence span withheld — anchor status: not_found]
