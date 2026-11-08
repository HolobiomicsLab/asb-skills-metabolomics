---
name: spectral-contrast-angle-computation
description: Use when you have peptide sequences with both observed and Prosit-predicted
  fragment-ion intensity vectors and need a similarity or sequence-stability feature
  for PSM rescoring.
when_to_use_negative:
- You only have search-engine scores or no predicted fragment-ion intensity vectors
  from Prosit — the SA and Prosit-delta require paired observed/predicted intensity
  vectors
- The goal is generic PSM rescoring from an existing feature table without spectrum-level
  similarity features — use ristretto, Percolator, or MS2Rescore directly instead
- The input is already an FDR-filtered final identification list with no raw spectra
  or prediction vectors available for recomputation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Prosit
  role: Provides the predicted fragment-ion intensity vectors against which the observed
    spectrum's spectral contrast angle is computed, including for adjacent-residue-swap
    variants
- name: inSPIRE
  role: Consumes the Prosit-delta spectral-angle stability feature for PSM rescoring
- name: Percolator
  role: Downstream semi-supervised PSM rescoring; the Prosit-delta score is conceptually
    related to Percolator's original ΔXCorr score
- name: MS2Rescore
  role: Alternative AI-assisted rescoring platform in which predicted fragment-ion
    intensity features of this kind improve PSM discrimination
  repo: https://github.com/compomics/ms2rescore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:5d5fe47936dabfc9def757286b868dace14c9d7c0f3f43bc22f924f3c12fcc43
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - MS2Rescore
  - Percolator
  - Prosit
  - inSPIRE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# spectral-contrast-angle-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the normalized spectral contrast angle (SA) between observed and Prosit-predicted fragment-ion intensity vectors, and derives the inSPIRE Prosit-delta stability score by swapping adjacent residues and recomputing the SA. Low Prosit-delta flags candidate peptide-spectrum matches at risk of being misassigned to similar but incorrect peptide sequences.

## When to use

Use when you have peptide sequences with both observed and Prosit-predicted fragment-ion intensity vectors and need a similarity or sequence-stability feature for PSM rescoring.

## When NOT to use

- You only have search-engine scores or no predicted fragment-ion intensity vectors from Prosit — the SA and Prosit-delta require paired observed/predicted intensity vectors
- The goal is generic PSM rescoring from an existing feature table without spectrum-level similarity features — use ristretto, Percolator, or MS2Rescore directly instead
- The input is already an FDR-filtered final identification list with no raw spectra or prediction vectors available for recomputation

## Inputs

- Peptide sequences with observed fragment-ion intensity vectors from acquired MS2 spectra
- Prosit-predicted fragment-ion intensity vectors for each candidate peptide sequence and its adjacent-residue-swap variants

## Outputs

- Per-peptide table of normalized spectral contrast angle (SA) values and Prosit-delta stability scores, e.g. prosit_delta_scores.csv

## How to apply

For each peptide, compute the normalized spectral contrast angle as the cosine of the angle between the observed and predicted intensity vectors: the dot product divided by the product of the L2 norms. Then, for each pair of adjacent amino-acid positions, swap the two residues, obtain the Prosit-predicted fragment-ion assignment for the swapped sequence, and recompute the SA against the observed vector. Aggregate the stability delta between the original SA and the adjacent-residue-swap SA values into a per-peptide Prosit-delta score. This is conceptually related to the ΔXCorr score originally used by Percolator, and low-delta sequences are more likely to be misassigned. Write a per-peptide table of SA values and stability deltas (e.g. prosit_delta_scores.csv) for use as a rescoring feature by tools such as inSPIRE.

## Related tools

- **Prosit** (Provides the predicted fragment-ion intensity vectors against which the observed spectrum's spectral contrast angle is computed, including for adjacent-residue-swap variants)
- **inSPIRE** (Consumes the Prosit-delta spectral-angle stability feature for PSM rescoring)
- **Percolator** (Downstream semi-supervised PSM rescoring; the Prosit-delta score is conceptually related to Percolator's original ΔXCorr score)
- **MS2Rescore** (Alternative AI-assisted rescoring platform in which predicted fragment-ion intensity features of this kind improve PSM discrimination) — https://github.com/compomics/ms2rescore

## Examples

```
SA = float(observed_vec @ predicted_vec / (np.linalg.norm(observed_vec) * np.linalg.norm(predicted_vec)))
```

## Evaluation signals

- Output table contains exactly one row per peptide in the worked example
- All computed SA values fall within [0, 1], since SA is a normalized cosine between non-negative intensity vectors
- All Prosit-delta scores are finite (no NaN/inf from zero-norm vectors)
- SA of the original (unswapped) sequence is recomputed independently and matches a direct cosine similarity check
- Swapped-sequence variants are generated for every adjacent residue pair, not a random subset

## Limitations

- Sequences with a low Prosit-delta are more likely to be misassigned to similar, though incorrect peptide sequences — the score flags instability rather than proving correctness
- Requires accurate Prosit predictions; incorrect predicted intensities for either the original or swapped sequences propagate into the delta
- Adjacent-residue swaps only probe local sequence ambiguity (isobaric/isomeric position swaps), not distant misassignments

## Evidence

- [other] The Prosit-delta score, used by inSPIRE, represents the stability of the normalized spectral contrast angle (SA) after two amino acid positions are switched: [evidence span withheld — anchor status: not_found]
- [other] sequences with a low Prosit-delta are more likely to be misassigned to similar, though incorrect peptide sequences: [evidence span withheld — anchor status: not_found]
- [abstract] PSM rescoring is a machine learning-based post-processing approach to better distinguish correct from incorrect PSMs: [evidence span withheld — anchor status: not_found]
