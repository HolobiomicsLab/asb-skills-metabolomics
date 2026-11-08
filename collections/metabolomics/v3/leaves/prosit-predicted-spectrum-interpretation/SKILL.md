---
name: prosit-predicted-spectrum-interpretation
description: Use when PSMs carry both an observed fragment-ion intensity vector and
  a Prosit/MS2PIP-predicted intensity vector, and you need a per-peptide confidence
  signal for immunopeptidomics-style rescoring.
when_to_use_negative:
- Input PSMs lack an observed fragment-ion intensity vector or a predicted intensity
  vector for the peptidoform's modification state, so no contrast angle can be computed
- Input is already a rescored feature table (e.g., ristretto/Percolator output with
  q-values); the angle/delta features would be redundant downstream features rather
  than the right entry point
- You need raw spectrum-level identification rather than interpretation of already-annotated
  candidate PSMs — run the search/PSM-generation stage first
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Prosit
  role: Provides the predicted fragment-ion intensity vectors used as the reference
    spectrum for the normalized spectral contrast angle and swap-variant recomputation
- name: MS2Rescore
  role: Feature-generation platform whose ms2pip component supplies predicted fragment-ion
    intensities that feed predicted-peptidoform-property rescoring
  repo: github.com/compomics/ms2rescore
- name: inSPIRE
  role: Immunopeptidomics rescoring tool that applies the Prosit-delta stability score
    as a PSM feature
- name: Percolator
  role: Semi-supervised rescoring framework whose original ΔXCorr concept the Prosit-delta
    score generalizes; consumes the generated features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:f5eae9f6701147faf8913fd3be35f1367e7c1f37464822528d108df72e398cc8
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# prosit-predicted-spectrum-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets peptide-spectrum matches (PSMs) against Prosit/MS2PIP-predicted fragment-ion intensity vectors using the normalized spectral contrast angle (SA) and its stability under adjacent-residue swaps (the Prosit-delta score, as used by inSPIRE), to flag candidate PSMs likely to be misassigned to a similar incorrect sequence.

## When to use

Use when PSMs carry both an observed fragment-ion intensity vector and a Prosit/MS2PIP-predicted intensity vector, and you need a per-peptide confidence signal for immunopeptidomics-style rescoring.

## When NOT to use

- Input PSMs lack an observed fragment-ion intensity vector or a predicted intensity vector for the peptidoform's modification state, so no contrast angle can be computed
- Input is already a rescored feature table (e.g., ristretto/Percolator output with q-values); the angle/delta features would be redundant downstream features rather than the right entry point
- You need raw spectrum-level identification rather than interpretation of already-annotated candidate PSMs — run the search/PSM-generation stage first

## Inputs

- Peptide sequences from a PSM list with observed fragment-ion intensity vectors (from the annotated spectrum)
- Prosit-predicted fragment-ion intensity vectors for the same peptidoforms (e.g., via the MS2PIP/ms2pip feature generator with an HCD model)
- A bundled worked example of peptides with paired observed and predicted intensity vectors, used for validation

## Outputs

- prosit_delta_scores.csv: one row per peptide, containing the spectral contrast angle (SA) and the Prosit-delta stability score
- Per-peptide table of original and adjacent-residue-swap SA variants used to derive the stability delta

## How to apply

For each peptide, compute the normalized spectral contrast angle (SA) between the observed and predicted fragment-ion intensity vectors as their cosine: the dot product divided by the product of the L2 norms, yielding a value in [0, 1]. Then, for every pair of adjacent amino-acid positions, swap the two residues, obtain the predicted fragment-ion assignment for the swapped sequence, and recompute the SA against the same observed vector. Combine the original SA with the swap-variant SA values into a stability delta — the Prosit-delta score, conceptually related to the ΔXCorr score originally used by Percolator — where a low Prosit-delta marks sequences more likely misassigned to a similar but incorrect peptide sequence. Collect the per-peptide SA values and stability deltas in a table and write it to prosit_delta_scores.csv; these features can then feed downstream semi-supervised rescoring tools such as ristretto/Percolator within an MS²Rescore-style pipeline.

## Related tools

- **Prosit** (Provides the predicted fragment-ion intensity vectors used as the reference spectrum for the normalized spectral contrast angle and swap-variant recomputation)
- **MS2Rescore** (Feature-generation platform whose ms2pip component supplies predicted fragment-ion intensities that feed predicted-peptidoform-property rescoring) — github.com/compomics/ms2rescore
- **inSPIRE** (Immunopeptidomics rescoring tool that applies the Prosit-delta stability score as a PSM feature)
- **Percolator** (Semi-supervised rescoring framework whose original ΔXCorr concept the Prosit-delta score generalizes; consumes the generated features)

## Examples

```
from scipy.spatial.distance import cosine; sa = 1 - cosine(observed, predicted)  # normalized spectral contrast angle per peptide
```

## Evaluation signals

- prosit_delta_scores.csv contains exactly one row per peptide in the worked example
- All SA values fall within [0, 1], consistent with a cosine-based normalized spectral contrast angle
- All Prosit-delta stability scores are finite (no NaN/inf), i.e., every adjacent swap produced a defined SA
- SA equals 1 - cosine(observed, predicted): verify dot(product of L2 norms) reproduction on a known peptide
- Sequences flagged low-delta should correspond to PSMs where a swapped sequence explains the spectrum nearly as well — spot-check against the swap-variant SA values

## Limitations

- A low Prosit-delta indicates elevated misassignment risk but does not by itself prove the PSM is wrong; it is a relative stability signal conceptually related to ΔXCorr
- The swap-based delta requires predicted intensities for every adjacent-residue-swap variant, so predictions must be available for each modified swapped sequence
- Predicted-peptidoform features (fragment ion intensities, retention time, collisional cross section) depend on the prediction model matching the acquisition style; the default ms2rescore ms2pip configuration uses the HCD model

## Evidence

- [other] The Prosit-delta score used by inSPIRE measures spectral-angle stability after residue swaps: [evidence span withheld — anchor status: not_found]
- [readme] MS²Rescore performs ultra-sensitive peptide identification rescoring with LC-MS predictors: [evidence span withheld — anchor status: external_doc]
