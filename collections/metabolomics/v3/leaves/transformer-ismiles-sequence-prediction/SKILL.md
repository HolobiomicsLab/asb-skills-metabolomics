---
name: transformer-ismiles-sequence-prediction
description: Use when you have a confirmed high-resolution precursor formula (two-decimal
  mass accuracy) from an MS2 spectrum and need the chemical structure (ISMILES) sequence
  for downstream fingerprint-based candidate ranking against PubChem.
when_to_use_negative:
- The precursor formula contains K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al, or a '.'
  separator (organic-compound-only scope) — excluded by the dataset filter rules.
- MS2 spectra lack high-resolution mass accuracy to two decimal places, violating
  the input quality filter.
- The ISMILES would exceed 130 characters or the SMILES cannot be standardized to
  ISMILES via PubChemPy.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_0091
tools:
- name: MSThunder
  role: Transformer model that interprets a precursor formula sequence into an ISMILES
    structure sequence
  repo: https://github.com/LQZ0123/MSThunder
- name: PubChemPy
  role: Retrieves candidate SMILES/ISMILES sequences from PubChem for the confirmed
    precursor formula, to compare against the predicted ISMILES
- name: RDKit
  role: Computes weighted molecular fingerprints of the predicted ISMILES and PubChem
    candidates for similarity ranking
- name: PyTorch (torch 1.13.0+cu116)
  role: Runtime for the transformer model implementing formula-to-ISMILES sequence
    prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.enceco.2025.07.022
    title: 'MSThunder: A deep learning-based nontargeted analytical framework for
      the accurate and rapid identification of unknown organic pollutants in water'
  source_content_hash: sha256:ec0e27a4ca0158a8b6f0707ae97736b162e208782d9e826e863dec569e9705af
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LQZ0123/MSThunder
  repo_url: https://github.com/LQZ0123/MSThunder
  tools:
  - MSConvert
  - MSThunder
  - PubChemPy
  - PyTorch (torch 1.13.0+cu116)
  - Python 3.10.5
  - RDKit
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# transformer-ismiles-sequence-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts an Isomeric SMILES (ISMILES) structure sequence directly from a precursor molecular formula using a transformer model (PyTorch 1.13.0+cu116), replacing library comparison and word-vector embedding of exact mass. This is the core prediction stage of the MSThunder pipeline for identifying unknown organic pollutants from MS2 spectra.

## When to use

Use when you have a confirmed high-resolution precursor formula (two-decimal mass accuracy) from an MS2 spectrum and need the chemical structure (ISMILES) sequence for downstream fingerprint-based candidate ranking against PubChem.

## When NOT to use

- The precursor formula contains K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al, or a '.' separator (organic-compound-only scope) — excluded by the dataset filter rules.
- MS2 spectra lack high-resolution mass accuracy to two decimal places, violating the input quality filter.
- The ISMILES would exceed 130 characters or the SMILES cannot be standardized to ISMILES via PubChemPy.

## Inputs

- Precursor molecular formula derived from a high-resolution MS2 spectrum (mass accuracy to two decimal places)
- Product ion formulas with isotopes extracted from the 50 highest-intensity MS2 peaks
- Batch-processed MS2 spectral file information (positive or negative ion mode)

## Outputs

- Predicted ISMILES (Isomeric SMILES) chemical structure sequence per precursor formula
- Per-spectrum prediction outcomes used for fingerprint similarity ranking of PubChem candidates

## How to apply

First extract product ion formulas with isotopes from the MS2 spectra (top 50 peaks by intensity) to build the input feature sequence, then feed the precursor-derived formula sequence into the transformer model — written in Python 3.10.5 with torch 1.13.0+cu116 — to interpret it into an ISMILES sequence character-by-character. The model bypasses conventional MS2 library comparison, transit prediction, and word-vector embedding of exact mass weight, because formula-to-structure mapping is less constrained by lack of training data than spectrum-to-structure. Input must have two-decimal-place mass accuracy; sequences longer than 130 characters, or formulas containing K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al, or '.' are excluded during dataset preparation. Predicted ISMILES is validated by re-generating candidate structures via PubChemPy retrieval and comparing weighted RDKit molecular fingerprints, with per-spectrum prediction time around 0.32 s. Judged correct when the top-1 ranked candidate matches the ground-truth structure on CASMI 2016 positive-mode spectra (52.8% direct-prediction, 59.1% top-1 accuracy).

## Related tools

- **MSThunder** (Transformer model that interprets a precursor formula sequence into an ISMILES structure sequence) — https://github.com/LQZ0123/MSThunder
- **PubChemPy** (Retrieves candidate SMILES/ISMILES sequences from PubChem for the confirmed precursor formula, to compare against the predicted ISMILES)
- **RDKit** (Computes weighted molecular fingerprints of the predicted ISMILES and PubChem candidates for similarity ranking)
- **PyTorch (torch 1.13.0+cu116)** (Runtime for the transformer model implementing formula-to-ISMILES sequence prediction)

## Examples

```
python msthunder.py --input Pesticides --ion-mode Positive --formula "C9H9ClO3"  # predict ISMILES for a confirmed precursor formula from a batch-processed MS2 file
```

## Evaluation signals

- Direct-prediction accuracy on the 127 CASMI 2016 positive-mode spectra reaches 52.8%
- Top-1 fingerprint-ranked candidate accuracy on CASMI 2016 reaches 59.1%
- Per-spectrum prediction time stays near the reported 0.32 s
- Every predicted ISMILES is ≤130 characters and contains no excluded elements (K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al) or '.'
- Predicted ISMILES is convertible to a valid SMILES via PubChemPy for PubChem retrieval

## Limitations

- The current release does not support offline processing of raw MS data; vendor raw files must be converted (e.g. via MSConvert) and returned by the authors before prediction.
- Dataset preparation filters out ISMILES containing K, Na, Au, Se, B, Fe, As, Ca, Mg, Cr, Al, or '.', SMILES not replaceable by ISMILES, and sequences over 130 characters — such compounds are outside model scope.
- Performance depends on availability of confirmed precursor formula; the workflow begins after formula confirmation, not from raw spectra.

## Evidence
