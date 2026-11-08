---
name: prediction-stability-delta-scoring
description: Use when rescoring immunopeptidomics or other PSMs with predicted fragment-ion
  intensities and you need to detect sequences whose spectral-angle fit is unstable
  under adjacent-residue swaps, as in inSPIRE-style rescoring.
when_to_use_negative:
- Input PSMs lack predicted fragment-ion intensities (no Prosit predictions available),
  since both the SA and the swapped-sequence SA require predicted intensity vectors
- The goal is generic PSM feature generation or semi-supervised rescoring (e.g. MS²Rescore's
  ms2pip/deeplc/ristretto pipeline) rather than specifically assessing sequence-assignment
  stability via spectral-angle deltas
- Peptides have no adjacent residue pairs worth swapping (e.g. very short sequences),
  making the stability delta uninformative
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Prosit
  role: Provides the predicted fragment-ion intensity vectors against which the observed
    spectrum's spectral contrast angle (SA) is computed, including for adjacent-residue-swap
    variants
- name: inSPIRE
  role: Immunopeptidomics rescoring tool that uses the Prosit-delta score as a stability
    feature
- name: Percolator
  role: Reference semi-supervised rescoring framework; Prosit-delta is conceptually
    related to Percolator's original ΔXCorr score
- name: MS2Rescore
  role: Rescoring platform context in which predicted-peptidoform-property features
    such as fragment ion intensities improve PSM identification
  repo: https://github.com/compomics/ms2rescore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:96daae572505b12312dc8cc8e5df3ea5de046307e02392aed29f133cdf8ea21f
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# prediction-stability-delta-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the Prosit-delta score: the stability of the normalized spectral contrast angle (SA) between observed and Prosit-predicted fragment-ion intensity vectors when adjacent amino-acid positions in a peptide are swapped. Low Prosit-delta flags peptide-spectrum matches likely misassigned to similar, though incorrect, peptide sequences.

## When to use

Use when rescoring immunopeptidomics or other PSMs with predicted fragment-ion intensities and you need to detect sequences whose spectral-angle fit is unstable under adjacent-residue swaps, as in inSPIRE-style rescoring.

## When NOT to use

- Input PSMs lack predicted fragment-ion intensities (no Prosit predictions available), since both the SA and the swapped-sequence SA require predicted intensity vectors
- The goal is generic PSM feature generation or semi-supervised rescoring (e.g. MS²Rescore's ms2pip/deeplc/ristretto pipeline) rather than specifically assessing sequence-assignment stability via spectral-angle deltas
- Peptides have no adjacent residue pairs worth swapping (e.g. very short sequences), making the stability delta uninformative

## Inputs

- Peptide sequences from a worked example or PSM list
- Observed fragment-ion intensity vectors from the spectra
- Prosit-predicted fragment-ion intensity vectors for each (and each swapped) sequence

## Outputs

- prosit_delta_scores.csv — one row per peptide, with the spectral contrast angle (SA) value and the stability delta score

## How to apply

For each peptide, compute the normalized spectral contrast angle (SA) between the observed fragment-ion intensity vector and the Prosit-predicted intensity vector as the cosine of the angle between them (dot product divided by the product of L2 norms). Then, for each pair of adjacent amino-acid positions, swap the two residues, obtain the predicted fragment-ion assignment for the swapped sequence, and recompute SA against the observed vector. The Prosit-delta score is the stability delta between the original SA and the SA values of the adjacent-residue-swap variants. This is conceptually related to the ΔXCorr score originally used by Percolator, and sequences with low Prosit-delta are more likely to be misassigned to similar, though incorrect, peptide sequences. Assemble a per-peptide table of SA values and stability deltas and write it to prosit_delta_scores.csv.

## Related tools

- **Prosit** (Provides the predicted fragment-ion intensity vectors against which the observed spectrum's spectral contrast angle (SA) is computed, including for adjacent-residue-swap variants)
- **inSPIRE** (Immunopeptidomics rescoring tool that uses the Prosit-delta score as a stability feature)
- **Percolator** (Reference semi-supervised rescoring framework; Prosit-delta is conceptually related to Percolator's original ΔXCorr score)
- **MS2Rescore** (Rescoring platform context in which predicted-peptidoform-property features such as fragment ion intensities improve PSM identification) — https://github.com/compomics/ms2rescore

## Examples

```
import numpy as np; sa = float(np.dot(obs, pred) / (np.linalg.norm(obs) * np.linalg.norm(pred))); delta = sa - max(sa_swapped); rows.to_csv('prosit_delta_scores.csv')
```

## Evaluation signals

- Output table prosit_delta_scores.csv contains exactly one row per peptide in the worked example
- All SA values fall within [0, 1], consistent with a cosine-based normalized spectral contrast angle
- All delta scores are finite (no NaN/inf from degenerate intensity vectors or zero L2 norms)
- Swapped-sequence SA values are recomputed against the same observed vector, so variants of the same peptide share a common reference spectrum

## Limitations

- Low Prosit-delta indicates higher likelihood of misassignment to similar but incorrect sequences; it is a stability indicator, not a standalone correctness verdict
- Requires accurate Prosit predictions for both the original and each swapped sequence, so prediction quality bounds the feature's informativeness
- Based on identifying immunopeptides, whose non-tryptic nature gives distinct spectral characteristics — behavior on tryptic data may differ

## Evidence

- [full_text] Prosit-delta definition and interpretation: [evidence span withheld — anchor status: not_found]
- [full_text] Relation to ΔXCorr: [evidence span withheld — anchor status: not_found]
- [readme] Rescoring context via MS²Rescore README: [evidence span withheld — anchor status: external_doc]
