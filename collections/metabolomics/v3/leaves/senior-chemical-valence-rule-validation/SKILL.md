---
name: senior-chemical-valence-rule-validation
description: Use when refining deep-learning formula predictions against a precursor
  mass and need to discard formulas that violate chemical valence constraints before
  ranking or reporting candidates.
when_to_use_negative:
- Input formulas have already been generated and validated by an external candidate
  generator such as SIRIUS or BUDDY and only need rescoring
- The task is adduct-type prediction or structural elucidation rather than filtering
  formula candidates for chemical validity
- Spectra contain m/z values above 1500, outside the m/z range (0, 1500] for which
  the refinement constraints were defined
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: FIDDLE
  role: Provides the breadth-first refinement (Algorithm 1) that edits atomic compositions
    and applies SENIOR rule filtering to align candidate formulas with the precursor
    mass
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI package exposing FIDDLE formula prediction and candidate generation from
    MGF files or Python arrays
  repo: https://github.com/josiehong/msfiddle
- name: SIRIUS
  role: Optional external generator of candidate formulas whose outputs can be incorporated
    into the same file-based workflow
  repo: https://github.com/boecker-lab/sirius
- name: BUDDY (msbuddy)
  role: Optional bottom-up candidate formula generator whose candidate lists can feed
    the same refinement/ranking pipeline
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:1a258d5ddc54101095d9977325b1b969d4c729a34c99052a3af3e89256061c26
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
  - BUDDY (msbuddy)
  - FIDDLE
  - SIRIUS
  - msfiddle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# SENIOR chemical valence rule validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters predicted molecular formulas by checking compliance with the SENIOR chemical-valence rules so that only chemically valid candidate formulas are retained downstream of deep-learning formula prediction (as in FIDDLE's post-processing).

## When to use

Use when refining deep-learning formula predictions against a precursor mass and need to discard formulas that violate chemical valence constraints before ranking or reporting candidates.

## When NOT to use

- Input formulas have already been generated and validated by an external candidate generator such as SIRIUS or BUDDY and only need rescoring
- The task is adduct-type prediction or structural elucidation rather than filtering formula candidates for chemical validity
- Spectra contain m/z values above 1500, outside the m/z range (0, 1500] for which the refinement constraints were defined

## Inputs

- Initial predicted molecular formulas from the FIDDLE deep-learning model
- Precursor m/z and precursor type parsed from the input MGF spectral file
- Instrument type (Q-TOF or Orbitrap) determining the mass tolerance

## Outputs

- Set of chemically valid candidate formulas (top K=5 per spectrum) satisfying SENIOR rules and precursor-mass tolerance
- Per-spectrum candidate formula lists written to candidate_formulas.csv

## How to apply

Take the raw predicted formulas from the FIDDLE deep-learning model and, during the breadth-first refinement step, make minimal adjustments to atom counts so each candidate formula complies with the SENIOR rules while aligning with the precursor mass within the instrument-specific tolerance (10 ppm for Q-TOF, 5 ppm for Orbitrap). Confinement to frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) and a total atom count cap of 300 bound the search space in which valence-valid formulas are sought. Surviving valence-compliant candidates are then ranked by the secondary confidence-scoring model, and the top K=5 formulas per spectrum are retained. Validation of the step means every returned candidate satisfies the SENIOR rules and falls within the ppm tolerance of the precursor mass.

## Related tools

- **FIDDLE** (Provides the breadth-first refinement (Algorithm 1) that edits atomic compositions and applies SENIOR rule filtering to align candidate formulas with the precursor mass) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI package exposing FIDDLE formula prediction and candidate generation from MGF files or Python arrays) — https://github.com/josiehong/msfiddle
- **SIRIUS** (Optional external generator of candidate formulas whose outputs can be incorporated into the same file-based workflow) — https://github.com/boecker-lab/sirius
- **BUDDY (msbuddy)** (Optional bottom-up candidate formula generator whose candidate lists can feed the same refinement/ranking pipeline) — https://github.com/Philipbear/msbuddy

## Examples

```
msfiddle-download-models
```

## Evaluation signals

- Every returned candidate formula satisfies the SENIOR chemical-valence rules
- Each candidate formula's mass falls within 10 ppm (Q-TOF) or 5 ppm (Orbitrap) of the precursor mass
- All candidates are composed only of frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) with total atom count ≤ 300
- Exactly K=5 candidate formulas are retained per spectrum and written to candidate_formulas.csv
- Top-5 results are consistent with FIDDLE's reported 93.6% top-5 accuracy benchmark on the evaluation set

## Limitations

- Valence validation is confined to formulas within the frequent-atom set and atom-count cap of 300, excluding rarer elements and larger molecules
- FIDDLE's accuracy varies across experimental conditions, especially those with less training data
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- The precursor-mass tolerance applies only to Q-TOF (10 ppm) and Orbitrap (5 ppm); other instrument types are not covered

## Evidence
