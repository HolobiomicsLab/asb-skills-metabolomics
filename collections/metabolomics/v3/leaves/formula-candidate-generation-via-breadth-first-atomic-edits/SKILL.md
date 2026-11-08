---
name: formula-candidate-generation-via-breadth-first-atomic-edits
description: Use when you have initial predicted formulas plus precursor m/z from
  an MGF file and need chemically valid candidate formulas within 10 ppm (Q-TOF) or
  5 ppm (Orbitrap) of the precursor mass.
when_to_use_negative:
- Input spectra have fewer than five peaks or precursor m/z outside the (0, 1500]
  range, where the underlying spectra are considered unreliable or out of scope
- The task is ab initio formula enumeration without initial deep-learning predictions
  — use bottom-up tools like BUDDY/msbuddy instead, since this skill refines existing
  predictions rather than searching the full formula space
- Molecules exceed 300 total atoms or contain elements outside the frequent-atom set
  (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), which are explicitly excluded
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FIDDLE
  role: Produces the initial deep-learning formula predictions that the breadth-first
    refinement edits into mass-aligned, SENIOR-compliant candidates
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI package providing the FIDDLE CLI and Python API for running prediction
    workflows, including BUDDY and SIRIUS candidate outputs in file-based workflows
  repo: https://github.com/josiehong/msfiddle
- name: BUDDY (msbuddy)
  role: Optional external source of candidate formula outputs that can be incorporated
    into the candidate generation workflow
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:23b1d305a547b2889b690732f47d21a91d1bc22bfbd0f04606e245ab02d2f6d0
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
  - msfiddle
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# formula-candidate-generation-via-breadth-first-atomic-edits

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines FIDDLE's initial deep-learning formula predictions into chemically valid candidates by a breadth-first search that minimally edits atomic compositions to match the precursor mass within instrument-specific ppm tolerance, then filters and ranks the results.

## When to use

Use when you have initial predicted formulas plus precursor m/z from an MGF file and need chemically valid candidate formulas within 10 ppm (Q-TOF) or 5 ppm (Orbitrap) of the precursor mass.

## When NOT to use

- Input spectra have fewer than five peaks or precursor m/z outside the (0, 1500] range, where the underlying spectra are considered unreliable or out of scope
- The task is ab initio formula enumeration without initial deep-learning predictions — use bottom-up tools like BUDDY/msbuddy instead, since this skill refines existing predictions rather than searching the full formula space
- Molecules exceed 300 total atoms or contain elements outside the frequent-atom set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), which are explicitly excluded

## Inputs

- MGF spectral file with initial FIDDLE-predicted formulas and fields 'precursor_mz' and 'precursor_type'
- Instrument type (Q-TOF or Orbitrap) determining the ppm mass tolerance
- Pre-trained FIDDLE model outputs (initial formula predictions)

## Outputs

- candidate_formulas.csv with per-spectrum lists of up to k=5 candidate formulas
- Chemically valid candidates satisfying SENIOR rules and the instrument-specific ppm tolerance

## How to apply

Parse the initial predicted formulas and precursor m/z values from the input MGF spectral file (fields 'precursor_mz', 'precursor_type'). Run the breadth-first search (Algorithm 1) that makes minimal edits to heavy-atom counts within a bounded search depth, aligning each candidate's mass to the precursor within 10 ppm for Q-TOF and 5 ppm for Orbitrap instruments. Filter out candidates violating the SENIOR chemical-valence rules, confine formulas to frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), and cap the total atom count at 300. Rank surviving candidates and retain the top K=5 formulas per spectrum, then write the per-spectrum candidate lists to candidate_formulas.csv. The rationale is that deep-learning predictions are close but not mass-exact, so minimal atomic edits suffice to reach validity while keeping the candidate space small.

## Related tools

- **FIDDLE** (Produces the initial deep-learning formula predictions that the breadth-first refinement edits into mass-aligned, SENIOR-compliant candidates) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI package providing the FIDDLE CLI and Python API for running prediction workflows, including BUDDY and SIRIUS candidate outputs in file-based workflows) — https://github.com/josiehong/msfiddle
- **BUDDY (msbuddy)** (Optional external source of candidate formula outputs that can be incorporated into the candidate generation workflow) — https://github.com/Philipbear/msbuddy

## Examples

```
msfiddle predict --input spectra.mgf --output candidate_formulas.csv
```

## Evaluation signals

- Every returned candidate formula satisfies the SENIOR chemical-valence rules
- Every candidate's neutral mass falls within the instrument-specific tolerance (10 ppm Q-TOF, 5 ppm Orbitrap) of the precursor mass
- At most K=5 candidate formulas are retained per spectrum in candidate_formulas.csv
- All candidate formulas are composed only of frequent atoms (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) with total atom count ≤ 300
- Module top-5 outputs are consistent with FIDDLE's reported 93.6% top-5 accuracy benchmark on the evaluation set

## Limitations

- Accuracy degrades with increasing precursor m/z because the candidate formula space grows exponentially
- Performance decreases on chimeric (co-eluting) spectra as spectral mixing increases
- Experimental-condition effects are not completely eliminated; accuracy varies across conditions, especially those with less training data

## Evidence
