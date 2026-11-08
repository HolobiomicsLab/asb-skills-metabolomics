---
name: unit-dalton-spectrum-normalization
description: Use when a model produces a probability distribution over molecular fragments
  with exact atomic-composition masses and you need to convert it into a final mass
  spectrum at 1 Da (or 0.1 Da) resolution.
when_to_use_negative:
- Input is already a measured or library spectrum (e.g. NIST-format spectrum) — no
  probability-weighted fragment summation is needed; use it only when reconstructing
  from a fragment probability distribution.
- Fragments carry only nominal (integer) masses without isotope/mass-defect information
  — the method requires exact masses computed with isotopic variability at natural
  abundance.
- The number of unique fragment formulae exceeds the RASSP filter cap (≤4096 unique
  fragment formulae, molecules ≤48 atoms for HCONFSPCl) — outside this regime the
  described enumeration/observation model was not validated.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: numpy
  role: Vectorized binning of probability-weighted fragment spectra into integer-Dalton
    bins and summing the weighted spectra
- name: RASSP (rassp-public)
  role: Reference implementation of the observation model — exact fragment spectrum
    computation, probability weighting, and summation
  repo: https://github.com/thejonaslab/rassp-public
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:65f881c59d341fe74b25297cf31b9f4cb4483fe668a06ecf1983d5bb593efcea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - RASSP (rassp-public)
  - numpy
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# unit-dalton-spectrum-normalization

## Summary

Reconstructs a predicted EI–MS spectrum by binning probability-weighted fragment mass spectra into integer-Dalton (or 0.10 Da) bins, mirroring RASSP's observation model that sums formula/subset spectra weighted by the model's output probability.

## When to use

Use when a model produces a probability distribution over molecular fragments with exact atomic-composition masses and you need to convert it into a final mass spectrum at 1 Da (or 0.1 Da) resolution.

## When NOT to use

- Input is already a measured or library spectrum (e.g. NIST-format spectrum) — no probability-weighted fragment summation is needed; use it only when reconstructing from a fragment probability distribution.
- Fragments carry only nominal (integer) masses without isotope/mass-defect information — the method requires exact masses computed with isotopic variability at natural abundance.
- The number of unique fragment formulae exceeds the RASSP filter cap (≤4096 unique fragment formulae, molecules ≤48 atoms for HCONFSPCl) — outside this regime the described enumeration/observation model was not validated.

## Inputs

- Enumerated fragment distribution (e.g. toluene fragments CSV) where each fragment is an atom subset or chemical subformula with model probability and exact atomic-composition mass
- Per-fragment elemental compositions supporting isotope-aware exact mass computation at natural abundance

## Outputs

- Final predicted 1 Da-resolution mass spectrum as (mass, intensity) rows, e.g. toluene_spectrum_1da.csv
- Final predicted 0.10 Da high-resolution mass spectrum as (mass, intensity) rows, e.g. toluene_spectrum_01da.csv

## How to apply

Load the enumerated fragment distribution so each fragment (atom subset or chemical subformula) carries its model probability and exact elemental composition. For each fragment, exactly compute the observed spectrum using summed atomic masses including isotopic variability at natural abundance and mass defect, then bin into integer-Dalton bins centered on integer values with unit widths (or bin without rounding at 0.10 Da resolution for the high-resolution variant). Weight each fragment's mass spectrum by the model's output probability and sum all observed mass spectra together to produce the final spectrum. Rationale: RASSP converts a predicted probability distribution over fragments into a spectrum precisely this way, enabling training and prediction at arbitrary resolution. The README confirms the output representation as a spectrum, i.e. a list of 2-tuple (mass, intensity) pairs, written to CSV columns mass, intensity.

## Related tools

- **numpy** (Vectorized binning of probability-weighted fragment spectra into integer-Dalton bins and summing the weighted spectra)
- **RASSP (rassp-public)** (Reference implementation of the observation model — exact fragment spectrum computation, probability weighting, and summation) — https://github.com/thejonaslab/rassp-public

## Examples

```
import numpy as np; frag = np.loadtxt('toluene_fragments.csv', delimiter=',', usecols=[1,2], skiprows=1); bins = np.arange(int(np.floor(frag[:,1].min())), int(np.ceil(frag[:,1].max()))+1); spec = np.zeros(len(bins)); probs, masses = frag[:,0], frag[:,1]; for p, m in zip(probs, masses): spec[np.argmin(np.abs(bins - np.floor(m + 0.5)))] += p; np.savetxt('toluene_spectrum_1da.csv', np.c_[bins, spec], delimiter=',', header='mass,intensity', comments='')
```

## Evaluation signals

- Mass conservation: at both 1 Da and 0.1 Da resolutions, the probability-weighted total spectral intensity equals the sum of fragment probabilities.
- All output intensities are non-negative; the summed spectrum is unit-L2-normalizable.
- 1 Da output bins are centered on integer Dalton values with unit widths; no spectral intensity appears at non-integer masses in the 1 Da variant.
- Output schema matches (mass, intensity) pairs as in the repo's spectrum representation.
- The 0.10 Da high-resolution variant shows sub-Dalton structure (multiple peaks per nominal-mass region) consistent with unrounded exact masses including mass defect.

## Limitations

- RASSP's GNNs consider only atoms, ignoring bond information beyond bond order, which limits fragment fidelity.
- More exotic fragmentation schemes were ignored in the enumeration process; their inclusion could potentially improve recall.
- NumPy versioning in the conda environment is brittle: Numba requires NumPy ≤1.21 and Tinygraph requires an older ndarray ABI, so numpy==1.21 must be pinned.

## Evidence

- [methods] Probability-weighted summation of fragment spectra as the core reconstruction step: "We then weight each formula/subset’s mass spectrum according to the model’s output probability and sum all of the observed mass spectra together"
- [methods] Isotope-aware exact observed spectrum per formula: "Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect."
- [abstract] Arbitrary-resolution training/prediction capability of both models: "Both models can train on and predict spectral data at arbitrary resolution"
- [readme] Spectrum file format as (mass, intensity) pairs: "A spectrum is represented as a list of 2-tuple (mass, intensity) pairs."
- [methods] Dataset filter scope bounding applicability (HCONFSPCl, ≤48 atoms, ≤4096 fragment formulae): "After filtering the dataset down to molecules containing only HCONFSPCl atoms, with total atoms ≤ 48, number of unique fragment formulae ≤ 4096"
