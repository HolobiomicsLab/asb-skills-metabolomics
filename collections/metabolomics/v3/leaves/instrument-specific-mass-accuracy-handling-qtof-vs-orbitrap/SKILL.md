---
name: instrument-specific-mass-accuracy-handling-qtof-vs-orbitrap
description: Use when refining formula candidates from MS/MS data whose precursor
  m/z and instrument type (Q-TOF or Orbitrap) are known, and candidate formulas must
  fall within the correct ppm tolerance of the precursor mass.
when_to_use_negative:
- Spectra lacking precursor_mz/precursor_type information or an unknown instrument
  type, so no ppm tolerance can be justified
- Formulas involving elements outside the frequent-atom set (C, H, O, N, F, S, Cl,
  P, B, I, Br, Na, K) or molecules exceeding 300 atoms
- Chimeric/co-eluting spectra where FIDDLE's performance is known to degrade, making
  tolerance-based refinement unreliable
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3292
tools:
- name: FIDDLE
  role: Performs the deep-learning prediction and breadth-first candidate refinement
    under the instrument-specific ppm tolerance
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI CLI/Python API that runs pre-trained FIDDLE models on MGF files with
    the same tolerance settings
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:a74442d08f4eebc490f0d114e713278f6456253def6eda922238defcee329f5c
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
  - msfiddle
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Instrument-specific mass accuracy handling (Q-TOF vs Orbitrap)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies instrument-dependent precursor mass tolerances (10 ppm for Q-TOF, 5 ppm for Orbitrap) when refining candidate molecular formulas so that FIDDLE's deep-learning predictions are constrained to formulas matching the measured precursor mass. This mirrors the accuracy profiles of the two instrument families used to train and evaluate FIDDLE.

## When to use

Use when refining formula candidates from MS/MS data whose precursor m/z and instrument type (Q-TOF or Orbitrap) are known, and candidate formulas must fall within the correct ppm tolerance of the precursor mass.

## When NOT to use

- Spectra lacking precursor_mz/precursor_type information or an unknown instrument type, so no ppm tolerance can be justified
- Formulas involving elements outside the frequent-atom set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) or molecules exceeding 300 atoms
- Chimeric/co-eluting spectra where FIDDLE's performance is known to degrade, making tolerance-based refinement unreliable

## Inputs

- MGF file with per-spectrum 'precursor_mz' and 'precursor_type' fields
- Instrument type annotation (Q-TOF or Orbitrap) for each spectrum or dataset
- Initial formula predictions from the FIDDLE deep-learning model

## Outputs

- Per-spectrum list of top K=5 candidate formulas within the instrument-specific ppm tolerance
- candidate_formulas.csv with refined candidate formula lists

## How to apply

Parse the precursor m/z and precursor type from the input MGF file, identify the instrument family, and set the mass tolerance to 10 ppm for Q-TOF data or 5 ppm for Orbitrap data. During FIDDLE's breadth-first refinement (Algorithm 1), reject any candidate formula whose computed mass deviates from the precursor mass by more than this instrument-specific tolerance. Simultaneously enforce the SENIOR chemical-valence rules, restrict formulas to the frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), and cap the total atom count at 300. Rank surviving candidates with the confidence-scoring model and retain the top K=5 formulas per spectrum; the tighter 5 ppm Orbitrap window is justified by that instrument's higher mass accuracy.

## Related tools

- **FIDDLE** (Performs the deep-learning prediction and breadth-first candidate refinement under the instrument-specific ppm tolerance) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI CLI/Python API that runs pre-trained FIDDLE models on MGF files with the same tolerance settings) — https://github.com/josiehong/msfiddle

## Examples

```
msfiddle-download-models
```

## Evaluation signals

- Every returned candidate formula satisfies the SENIOR chemical-valence rules
- All candidates fall within 10 ppm (Q-TOF) or 5 ppm (Orbitrap) of the precursor mass
- Exactly up to K=5 candidate formulas are emitted per spectrum
- All candidate formulas contain only the frequent atoms and have a total atom count <= 300
- Top-5 outputs are consistent with FIDDLE's reported 93.6% top-5 accuracy benchmark on the evaluation set

## Limitations

- Accuracy varies across experimental conditions, especially those with less training data
- Performance decreases on chimeric (co-eluting) spectra as spectral mixing increases
- FDR control via confidence score thresholds is proposed but not implemented

## Evidence

- [readme] msfiddle runtime: [evidence span withheld — anchor status: external_doc]
