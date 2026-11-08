---
name: fragment-ion-intensity-vector-comparison
description: 'Use when candidate PSMs must be distinguished from near-identical sequence
  variants: e.g., scoring the stability of the spectral angle under adjacent-residue
  swaps using Prosit-predicted fragment-ion intensities in immunopeptidomics rescoring.'
when_to_use_negative:
- No Prosit-predicted fragment-ion intensities are available for the peptide or its
  swapped variants, so the spectral contrast angle cannot be computed
- The goal is to discriminate PSMs from unrelated peptides rather than to resolve
  near-identical, isobaric sequence variants that the swap-based stability delta targets
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Prosit
  role: Predicts fragment-ion intensity vectors for each peptide and for adjacent-residue-swap
    variants used in the spectral-angle stability computation
- name: inSPIRE
  role: PSM rescoring tool that consumes the Prosit-delta spectral-angle stability
    feature for immunopeptidomics
- name: MS2Rescore
  role: Rescoring platform whose ms2pip feature generator compares predicted vs. observed
    fragment ion intensities in the same spirit
  repo: https://github.com/compomics/ms2rescore
- name: Percolator
  role: Semi-supervised PSM rescoring whose ΔXCorr score is the conceptual predecessor
    of the Prosit-delta stability delta
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:dfb415b8e273d0eb03e0726a5bbbd850fe67daf58ca365b81ca32a97cf9fd0b5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# fragment-ion-intensity-vector-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the normalized spectral contrast angle (SA) between observed and Prosit-predicted fragment-ion intensity vectors and its Prosit-delta stability score, as used by inSPIRE, to flag PSMs likely misassigned to similar but incorrect peptide sequences.

## When to use

Use when candidate PSMs must be distinguished from near-identical sequence variants: e.g., scoring the stability of the spectral angle under adjacent-residue swaps using Prosit-predicted fragment-ion intensities in immunopeptidomics rescoring.

## When NOT to use

- No Prosit-predicted fragment-ion intensities are available for the peptide or its swapped variants, so the spectral contrast angle cannot be computed
- The goal is to discriminate PSMs from unrelated peptides rather than to resolve near-identical, isobaric sequence variants that the swap-based stability delta targets

## Inputs

- Peptide sequences with observed fragment-ion intensity vectors (from PSMs/spectra, e.g. mzid or tsv PSM files)
- Prosit-predicted fragment-ion intensity vectors for each peptide and for adjacent-residue-swap variants

## Outputs

- prosit_delta_scores.csv — per-peptide table of normalized spectral contrast angle (SA) values and Prosit-delta stability scores

## How to apply

Load the worked example with peptide sequences and their observed plus Prosit-predicted fragment-ion intensity vectors. For each peptide, compute the normalized spectral contrast angle (SA) as the cosine of the angle between observed and predicted intensity vectors (dot product divided by the product of L2 norms), yielding values in [0, 1]. For each pair of adjacent amino-acid positions, swap the two residues, obtain the predicted fragment-ion intensities for the swapped sequence, and recompute the SA against the same observed vector. Derive the Prosit-delta score as the stability delta between the original SA and the adjacent-residue-swap SA values — conceptually related to Percolator's ΔXCorr. Sequences with a low Prosit-delta are more likely to be misassigned to similar, though incorrect peptide sequences, so the feature feeds downstream PSM rescoring (e.g., in inSPIRE or MS²Rescore-style semi-supervised learners such as ristretto). Assemble the per-peptide table of SA values and stability deltas and write it to prosit_delta_scores.csv.

## Related tools

- **Prosit** (Predicts fragment-ion intensity vectors for each peptide and for adjacent-residue-swap variants used in the spectral-angle stability computation)
- **inSPIRE** (PSM rescoring tool that consumes the Prosit-delta spectral-angle stability feature for immunopeptidomics)
- **MS2Rescore** (Rescoring platform whose ms2pip feature generator compares predicted vs. observed fragment ion intensities in the same spirit) — https://github.com/compomics/ms2rescore
- **Percolator** (Semi-supervised PSM rescoring whose ΔXCorr score is the conceptual predecessor of the Prosit-delta stability delta)

## Examples

```
from numpy import dot, linalg; sa = dot(observed, predicted) / (linalg.norm(observed) * linalg.norm(predicted))
```

## Evaluation signals

- Output table contains exactly one row per peptide in the worked example
- All spectral contrast angle (SA) values fall within [0, 1]
- All Prosit-delta stability scores are finite (no NaN/inf)
- Low Prosit-delta values should preferentially co-occur with PSMs later retracted or misassigned to similar incorrect sequences

## Limitations

- A low Prosit-delta indicates the SA is unstable under adjacent-residue swaps, so the PSM is more likely misassigned to a similar, though incorrect, peptide sequence
- The stability assessment covers adjacent two-residue swaps only; non-adjacent or multi-residue confusions are not probed
- Requires a Prosit prediction for every swapped sequence, multiplying prediction calls by the number of adjacent residue pairs per peptide

## Evidence

- [full_text] The Prosit-delta score, used by inSPIRE, represents the stability of the normalized spectral contrast angle (SA) after two amino acid positions are switched: [evidence span withheld — anchor status: not_found]
- [full_text] sequences with a low Prosit-delta are more likely to be misassigned to similar, though incorrect peptide sequences: [evidence span withheld — anchor status: not_found]
- [abstract] PSM rescoring has emerged as a machine learning-based solution to address challenges in mass spectrometry-based immunopeptidomics data analysis: [evidence span withheld — anchor status: not_found]
- [abstract] features based on predicted peptidoform properties, including fragment ion intensities, retention time, and collisional cross section, have been used to improve the accuracy and sensitivity of: "features based on predicted peptidoform properties, including fragment ion intensities, retention time, and collisional cross section, have been used to improve the accuracy and sensitivity of"
