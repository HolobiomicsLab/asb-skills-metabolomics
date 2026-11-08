---
name: mass-bin-accuracy-profiling
description: Use when evaluating a formula-prediction model's per-spectrum predictions
  on a held-out MS/MS test set and you need accuracy stratified by precursor mass,
  LogP polarity, or chemical superclass rather than a single aggregate score.
when_to_use_negative:
- Spectra with precursor m/z above 1500, which are excluded from FIDDLE's scope due
  to rarity
- Compounds containing atoms outside the frequent-atom set (C, H, O, N, F, S, Cl,
  P, B, I, Br, Na, K) or exceeding 300 total atoms
- Chimeric (co-eluting) spectra evaluation — FIDDLE's performance there is a separate
  analysis, and adduct-type stratification is not supported since adduct prediction
  is left for future work
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Deep learning model (TCN and rescore checkpoints, Q-TOF and Orbitrap variants)
    whose test-set predictions are stratified and evaluated
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI package/CLI for running FIDDLE predictions from MGF files to produce
    the per-spectrum prediction outputs
  repo: https://github.com/josiehong/msfiddle
- name: Q-TOF
  role: Instrument type used as a stratification variable in the accuracy profiling
- name: Orbitrap
  role: Instrument type used as a stratification variable in the accuracy profiling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:291d3a74bcf14327c3786aa982d1afecec94122ae664d4f31301e0edffa94a29
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
  - FIDDLE
  - Orbitrap
  - Q-TOF
  - msfiddle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Mass-bin accuracy profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies FIDDLE's top-5 formula identification accuracy on the NIST23 test set by precursor molecular mass bins (e.g., <200 Da vs. larger), LogP polarity groups, and ClassyFire superclass, for Q-TOF and Orbitrap instruments separately.

## When to use

Use when evaluating a formula-prediction model's per-spectrum predictions on a held-out MS/MS test set and you need accuracy stratified by precursor mass, LogP polarity, or chemical superclass rather than a single aggregate score.

## When NOT to use

- Spectra with precursor m/z above 1500, which are excluded from FIDDLE's scope due to rarity
- Compounds containing atoms outside the frequent-atom set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) or exceeding 300 total atoms
- Chimeric (co-eluting) spectra evaluation — FIDDLE's performance there is a separate analysis, and adduct-type stratification is not supported since adduct prediction is left for future work

## Inputs

- Per-spectrum top-5 formula predictions CSV from FIDDLE on the NIST23 test set (Q-TOF and/or Orbitrap spectra)
- Ground-truth molecular formulas for test compounds
- Canonical SMILES-derived LogP values for test compounds
- ClassyFire superclass annotations for test compounds
- Precursor m/z and instrument type per spectrum

## Outputs

- Stratified table of top-5 accuracy by mass bin (e.g., <200 Da vs. larger) per instrument type
- Top-5 accuracy by LogP polarity group (hydrophilic, moderate, lipophilic)
- Top-5 accuracy by ClassyFire superclass with underperforming superclasses flagged
- Fig. 3c/d/f-style summary panels showing mass-bin, LogP-trend, and superclass profiles

## How to apply

Run the released FIDDLE TCN and rescore checkpoints (Q-TOF and Orbitrap variants) over held-out test spectra via the repository's NIST23 evaluation script (experiments_test_nist23.sh), writing per-spectrum predictions to a results CSV. Parse the CSV and join each spectrum's ground-truth formula with molecular annotations derived from canonical SMILES: cLogP for polarity binning and ClassyFire superclass labels. Bin spectra into mass strata (e.g., <200 Da vs. larger) and LogP polarity groups (hydrophilic/moderate/lipophilic), compute top-5 accuracy per stratum per instrument, and flag underperforming superclasses (organohalogens, polymers, organosulfur, unknown). The rationale is that accuracy varies systematically with molecular properties: precursor m/z range is confined to (0, 1500], and the paper reports strong property-dependent accuracy differences that aggregate metrics hide.

## Related tools

- **FIDDLE** (Deep learning model (TCN and rescore checkpoints, Q-TOF and Orbitrap variants) whose test-set predictions are stratified and evaluated) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI package/CLI for running FIDDLE predictions from MGF files to produce the per-spectrum prediction outputs) — https://github.com/josiehong/msfiddle
- **Q-TOF** (Instrument type used as a stratification variable in the accuracy profiling)
- **Orbitrap** (Instrument type used as a stratification variable in the accuracy profiling)

## Examples

```
bash experiments_test_nist23.sh  # run FIDDLE Q-TOF/Orbitrap checkpoints on NIST23 test spectra, writing per-spectrum top-5 predictions to a results CSV
```

## Evaluation signals

- Reproduces the reported <200 Da top-5 accuracy values: 97.3% (Q-TOF) and 95.0% (Orbitrap)
- Polarity trend matches the reported ordering — polar 95.0% > moderate 94.7% > nonpolar 88.8% — a ~6.2% gap consistent across Q-TOF and Orbitrap
- Flags the paper's underperforming superclass list: organohalogens, polymers, organosulfur, and unknown
- Aggregate top-5 accuracy across all strata averages to the reported 93.6% (with top-1 at 88.3%)
- Every test spectrum appears in exactly one mass bin and one polarity group; strata sizes sum to the full test set size

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data; the property-dependent performance gap is not fully eliminated
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases, so mass-bin profiling on mixed spectra is unreliable
- Confidence-score-threshold FDR control is proposed but not implemented, so stratum-level accuracy cannot be FDR-adjusted in this workflow

## Evidence

- [results] Reported polarity-gap finding: [evidence span withheld — anchor status: not_found]
