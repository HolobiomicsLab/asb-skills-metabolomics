---
name: peptide-sequence-adjacent-swap-perturbation
description: Use when rescoring peptide-spectrum matches with predicted-fragment-intensity
  features and you need a stability feature that distinguishes a peptide from its
  adjacent-residue-swap neighbours.
when_to_use_negative:
- No Prosit (or other) predicted fragment-ion intensities are available for the original
  or swapped peptide sequences — the contrast angle cannot be computed
- Input is already a rescoring feature table being consumed by a semi-supervised model
  (e.g. ristretto/Percolator) — this skill generates a feature, it does not rescore
- The PSM set is not being re-ranked and only final FDR-filtered identifications are
  needed
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Prosit
  role: Predicts fragment-ion intensities for the original peptide and each adjacent-residue-swap
    variant, enabling recomputation of the spectral contrast angle
- name: inSPIRE
  role: Immunopeptidomics rescoring tool that uses the Prosit-delta stability score
    as a PSM-rescoring feature
- name: Percolator
  role: Original semi-supervised rescoring engine whose ΔXCorr score is the conceptual
    predecessor of the Prosit-delta feature
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:466433fadb33c7348becdfd5cb144ed87a61c92a0b4f295e0710a2c3d8441f59
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  tools:
  - MS2Rescore
  - Percolator
  - Prosit
  - inSPIRE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# peptide-sequence-adjacent-swap-perturbation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the Prosit-delta score used by inSPIRE: the stability of the normalized spectral contrast angle (SA) between observed and Prosit-predicted fragment-ion intensities after swapping adjacent amino acid positions in a candidate peptide. Low Prosit-delta flags PSMs likely misassigned to similar but incorrect peptide sequences, in the same spirit as Percolator's original ΔXCorr feature.

## When to use

Use when rescoring peptide-spectrum matches with predicted-fragment-intensity features and you need a stability feature that distinguishes a peptide from its adjacent-residue-swap neighbours.

## When NOT to use

- No Prosit (or other) predicted fragment-ion intensities are available for the original or swapped peptide sequences — the contrast angle cannot be computed
- Input is already a rescoring feature table being consumed by a semi-supervised model (e.g. ristretto/Percolator) — this skill generates a feature, it does not rescore
- The PSM set is not being re-ranked and only final FDR-filtered identifications are needed

## Inputs

- Peptide sequences from candidate PSMs (worked example bundled with the skill)
- Observed fragment-ion intensity vectors from the corresponding spectra
- Prosit-predicted fragment-ion intensity vectors for each original and adjacent-residue-swap peptide sequence

## Outputs

- prosit_delta_scores.csv — per-peptide table of normalized spectral contrast angle (SA) values and stability delta scores

## How to apply

For each candidate peptide, load its observed fragment-ion intensity vector and the Prosit-predicted intensity vector, then compute the normalized spectral contrast angle (SA) as the dot product divided by the product of L2 norms of the two vectors. For each pair of adjacent amino-acid positions, swap the two residues, obtain the Prosit-predicted fragment-ion assignment for the swapped sequence, and recompute SA against the same observed vector. The Prosit-delta score is the stability delta between the original SA and the SA values of the adjacent-residue-swap variants; sequences with a low Prosit-delta are more likely misassigned to similar, incorrect peptide sequences. Assemble a per-peptide table of SA values and stability deltas and write it to prosit_delta_scores.csv. This feature can then be fed into a semi-supervised rescoring pipeline (e.g. inSPIRE, or Percolator-style tools such as ristretto with train_fdr≈0.01 and svm or lda models) alongside other predicted-property features like retention time.

## Related tools

- **Prosit** (Predicts fragment-ion intensities for the original peptide and each adjacent-residue-swap variant, enabling recomputation of the spectral contrast angle)
- **inSPIRE** (Immunopeptidomics rescoring tool that uses the Prosit-delta stability score as a PSM-rescoring feature)
- **Percolator** (Original semi-supervised rescoring engine whose ΔXCorr score is the conceptual predecessor of the Prosit-delta feature)

## Examples

```
import numpy as np; sa = obs @ pred / (np.linalg.norm(obs) * np.linalg.norm(pred))
```

## Evaluation signals

- prosit_delta_scores.csv contains exactly one row per peptide in the worked example
- All SA values fall within [0, 1] (they are cosines of the angle between observed and predicted intensity vectors)
- All delta scores are finite (no NaN/inf from zero-norm intensity vectors)
- Each peptide's delta reflects the spread between the original SA and the SA values of all adjacent-residue-swap variants

## Limitations

- Requires a Prosit prediction call per swapped sequence, so cost scales with peptide length (one variant per adjacent residue pair)
- A low Prosit-delta indicates elevated misassignment risk to a similar but incorrect sequence — it is a risk indicator, not a proof of incorrectness
- The feature is conceptually tied to predicted fragment-ion intensity quality; prediction errors for unusual (e.g. non-tryptic immunopeptide) sequences can distort the delta

## Evidence

- [other] Prosit-delta score definition: [evidence span withheld — anchor status: not_found]
