---
name: precursor-mass-stratified-performance-analysis
description: Use when benchmarking formula identification tools on tandem MS spectra
  and you need to quantify how accuracy degrades with increasing precursor m/z, especially
  for compounds above 800 Da.
when_to_use_negative:
- Input spectra are training data or overlap the training set — this skill is for
  held-out evaluation sets split by molecular canonical SMILES
- Spectra contain peaks or precursors with m/z above 1500, which are outside the m/z
  range (0, 1500] the models were evaluated on
- The goal is qualitative formula prediction for new unknowns rather than controlled
  method comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIDDLE
  role: Deep learning formula predictor whose top-1/top-5 accuracy is stratified by
    precursor mass; run via run_fiddle.py with instrument-matched checkpoints and
    optional --buddy_path/--sirius_path for the integrated variant
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Top-down baseline formula identifier (v6.0.1) run on the same NIST23-only
    spectra; output organized as sirius_output.csv for mass-stratified comparison
  repo: https://github.com/sirius-ms/sirius
- name: BUDDY
  role: Bottom-up baseline formula identifier (v0.3.6, msbuddy) run on the same spectra;
    output organized as buddy_output.csv for mass-stratified comparison
  repo: https://github.com/Philipbear/msbuddy
- name: msfiddle
  role: PyPI package / CLI providing pre-trained FIDDLE checkpoints (msfiddle-download-models)
    and file-based prediction workflows that can incorporate BUDDY and SIRIUS candidate
    outputs
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:439d285866a307563b091ed8d88b3ede495fe4766acb4df08d639ea4f096a375
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - Q-TOF
  - SIRIUS
  - msfiddle
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# precursor-mass-stratified-performance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates molecular formula identification accuracy (top-1, top-5, higher-K) separately for large compounds (molecular weight > 800 Da) versus smaller precursors, replicating the mass-stratified breakdown (e.g., Fig. 2) used to benchmark FIDDLE against SIRIUS and BUDDY on NIST23-only test spectra.

## When to use

Use when benchmarking formula identification tools on tandem MS spectra and you need to quantify how accuracy degrades with increasing precursor m/z, especially for compounds above 800 Da.

## When NOT to use

- Input spectra are training data or overlap the training set — this skill is for held-out evaluation sets split by molecular canonical SMILES
- Spectra contain peaks or precursors with m/z above 1500, which are outside the m/z range (0, 1500] the models were evaluated on
- The goal is qualitative formula prediction for new unknowns rather than controlled method comparison

## Inputs

- Held-out test MS/MS spectra in mgf format with title, precursor_mz, precursor_type, and collision_energy fields (e.g., NIST23-only test set)
- FIDDLE prediction outputs (run_fiddle.py with instrument-matched config, fiddle_tcn_orbitrap.pt / fiddle_rescore_orbitrap.pt or fiddle_tcn_qtof.pt / fiddle_rescore_qtof.pt checkpoints)
- Baseline outputs: sirius_output.csv (SIRIUS v6.0.1) and buddy_output.csv (BUDDY v0.3.6)
- Ground-truth molecular formulas and precursor molecular weights for each test compound

## Outputs

- Top-1, top-5, and higher-K accuracy per instrument type (Orbitrap, Q-TOF) and per method (FIDDLE, SIRIUS, BUDDY)
- Accuracy stratified by precursor mass, with a >800 Da stratum as in Fig. 2
- Reproduction check against reported baselines (~88.3% top-1, ~93.6% top-5)

## How to apply

Prepare the held-out test spectra (e.g., NIST23-only, in mgf format with title, precursor_mz, precursor_type, and collision_energy fields), run each predictor (FIDDLE via run_fiddle.py with the instrument-matched Orbitrap or Q-TOF config and checkpoints, plus SIRIUS v6.0.1 and BUDDY v0.3.6 with outputs organized as sirius_output.csv and buddy_output.csv). Compute top-1, top-5, and higher-K accuracies per instrument type, then partition predictions by precursor molecular weight, isolating a stratum for compounds above 800 Da as in Fig. 2. Apply instrument-appropriate precursor mass tolerances (10 ppm for Q-TOF, 5 ppm for Orbitrap) when judging a predicted formula correct. Validate the pipeline by confirming reproduction of the reported ~88.3% top-1 and ~93.6% top-5 accuracy on the full evaluation set, with FIDDLE outperforming SIRIUS and BUDDY by over 10%, and expect the >800 Da stratum to show degradation relative to small precursors — FIDDLE reported 0.642 (Q-TOF) and 0.813 (Orbitrap) top-1 accuracy on large compounds.

## Related tools

- **FIDDLE** (Deep learning formula predictor whose top-1/top-5 accuracy is stratified by precursor mass; run via run_fiddle.py with instrument-matched checkpoints and optional --buddy_path/--sirius_path for the integrated variant) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Top-down baseline formula identifier (v6.0.1) run on the same NIST23-only spectra; output organized as sirius_output.csv for mass-stratified comparison) — https://github.com/sirius-ms/sirius
- **BUDDY** (Bottom-up baseline formula identifier (v0.3.6, msbuddy) run on the same spectra; output organized as buddy_output.csv for mass-stratified comparison) — https://github.com/Philipbear/msbuddy
- **msfiddle** (PyPI package / CLI providing pre-trained FIDDLE checkpoints (msfiddle-download-models) and file-based prediction workflows that can incorporate BUDDY and SIRIUS candidate outputs) — https://github.com/josiehong/msfiddle

## Examples

```
bash running_scripts/experiments_test_nist23.sh  # runs run_fiddle.py with instrument-matched config/checkpoints and --buddy_path/--sirius_path, then compute top-K accuracy per instrument and >800 Da stratum
```

## Evaluation signals

- Full-evaluation-set top-1 accuracy reproduces ~88.3% and top-5 accuracy ~93.6%
- FIDDLE outperforms SIRIUS and BUDDY by over 10% on the evaluation set
- The >800 Da stratum shows degraded accuracy relative to small-precursor strata (FIDDLE top-1: 0.642 Q-TOF, 0.813 Orbitrap)
- A predicted formula counts as correct only within precursor mass tolerance of 10 ppm (Q-TOF) or 5 ppm (Orbitrap)
- Accuracy is reported separately per instrument type (Orbitrap vs Q-TOF) and for each compared method

## Limitations

- Accuracy decreases significantly with increasing precursor m/z due to the exponentially growing number of candidate molecular formulas
- Experimental condition effects are not completely eliminated; accuracy varies across conditions with less training data
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases

## Evidence
