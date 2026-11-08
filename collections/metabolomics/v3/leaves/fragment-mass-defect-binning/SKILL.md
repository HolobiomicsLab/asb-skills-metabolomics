---
name: fragment-mass-defect-binning
description: Use when a model outputs probabilities over chemical subformulae or atom
  subsets (fragment distribution with exact atomic-composition masses) and these must
  be decoded into a probability-weighted mass spectrum at a chosen mass resolution
  (e.g. 1 Da or 0.1 Da).
when_to_use_negative:
- The model output is already a finished (mass, intensity) spectrum — this skill is
  only the decoding/observation step, not a re-prediction of spectra from spectra.
- Molecules contain elements outside the HCONFSPCl scope or exceed 48 atoms / 4096
  unique fragment formulae — the RASSP fragment enumeration feeding this binning step
  was filtered to that scope.
- An exotic fragmentation scheme or bond-information-aware fragmentation is needed
  — RASSP's enumeration ignores bond information beyond the bond-breaking depth d=3
  and hydrogen rearrangements.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: RASSP
  role: Provides the msutil module ('Fast code for computing mass spectra and molecule
    subsets') that performs the isotope-aware exact spectrum computation and mass
    spectrum binning/summing; runs the forward inference whose fragment probabilities
    are binned here.
  repo: https://github.com/thejonaslab/rassp-public
- name: numpy
  role: Used for the numerical mass computation, binning (rounding to integer Dalton
    or 0.10 Da bins), probability weighting, and summation of fragment spectra; must
    be pinned to numpy==1.21 per README compatibility notes.
  repo: https://github.com/thejonaslab/rassp-public
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:aca5fa9ad63a7fb5a2ab099f4eaa748b5642737f5a4feb1c47f17ca098aad4cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - RASSP
  - numpy
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fragment-mass-defect-binning

## Summary

Converts a deep learning model's predicted probability distribution over molecular fragments with known exact masses into a predicted mass spectrum by computing each fragment's exact observed spectrum (isotope-aware, mass-defect accounting), binning it into integer- or sub-Dalton mass bins, weighting by fragment probability, and summing. This is the final observation/decoding step of the RASSP EI–MS prediction pipeline, producing spectra at 1 Da or 0.1 Da resolution.

## When to use

Use when a model outputs probabilities over chemical subformulae or atom subsets (fragment distribution with exact atomic-composition masses) and these must be decoded into a probability-weighted mass spectrum at a chosen mass resolution (e.g. 1 Da or 0.1 Da).

## When NOT to use

- The model output is already a finished (mass, intensity) spectrum — this skill is only the decoding/observation step, not a re-prediction of spectra from spectra.
- Molecules contain elements outside the HCONFSPCl scope or exceed 48 atoms / 4096 unique fragment formulae — the RASSP fragment enumeration feeding this binning step was filtered to that scope.
- An exotic fragmentation scheme or bond-information-aware fragmentation is needed — RASSP's enumeration ignores bond information beyond the bond-breaking depth d=3 and hydrogen rearrangements.

## Inputs

- Enumerated fragment distribution (CSV/parquet) mapping each fragment (atom subset or chemical subformula of the input molecule, e.g. toluene) to a model output probability and exact atomic-composition masses
- Target mass resolution (1 Da for tolulene_spectrum_1da.csv; 0.10 Da for toluene_spectrum_01da.csv)
- Element isotope masses and natural abundances for atoms present (HCONFSPCl scope)

## Outputs

- Probability-weighted summed mass spectrum as (mass, intensity) pairs — 1 Da-resolution spectrum (tolulene_spectrum_1da.csv) and/or 0.1 Da-resolution spectrum (toluene_spectrum_01da.csv)
- Per-fragment exact observed spectra (intermediate) incorporating isotope natural abundance and mass defect

## How to apply

Load the enumerated fragment distribution where each fragment (chemical subformula or atom subset) carries its model output probability and exact atomic-composition masses. For each fragment, compute the exact observed spectrum by summing atomic masses including isotopic variability at natural abundance and mass defect. Bin each fragment's spectrum into mass bins — for 1 Da output use bins centered on integer values with unit widths, rounded to integer Dalton; for high-resolution output repeat the exact mass computation without rounding and bin at 0.10 Da. Weight each fragment's binned spectrum by its model output probability and sum all weighted spectra together, so that the total intensity mass-conserves to the sum of fragment probabilities. Write the result as (mass, intensity) pairs matching the RASSP spectrum format List[Tuple[Float, Float]].

## Related tools

- **RASSP** (Provides the msutil module ('Fast code for computing mass spectra and molecule subsets') that performs the isotope-aware exact spectrum computation and mass spectrum binning/summing; runs the forward inference whose fragment probabilities are binned here.) — https://github.com/thejonaslab/rassp-public
- **numpy** (Used for the numerical mass computation, binning (rounding to integer Dalton or 0.10 Da bins), probability weighting, and summation of fragment spectra; must be pinned to numpy==1.21 per README compatibility notes.) — https://github.com/thejonaslab/rassp-public

## Examples

```
import numpy as np; from rassp.msutil import compute_exact_spectrum_and_bin; spect = sum(p * compute_exact_spectrum_and_bin(formula, bin_width_da=1.0) for formula, p in fragment_probs.items())
```

## Evaluation signals

- Mass conservation: at each resolution, the probability-weighted total spectral intensity equals the sum of fragment probabilities (weights sum to 1 over the fragment distribution).
- All output spectrum intensities are non-negative; the summed spectrum can be unit-L2-normalized (normalize_pred: True in the RASSP forward eval config).
- Bin alignment: 1 Da output bins are centered on integer Dalton values with unit widths; the 0.1 Da variant uses unrounded exact masses so the mass-defect structure is retained.
- Output spectra serialize as List[Tuple[Float, Float]] (mass, intensity) pairs, matching the `spect` column schema of RASSP parquet/sqlite outputs (e.g. forward.preds/<exp>.spect.sqlite).
- Spectral quality can be compared to ground truth via Stein/weighted dot product (SDP) using rassp's metrics.py, where FormulaNet reaches 92.9% and SubsetNet 91.8%.

## Limitations

- The observation model assumes fragments are generated only by recursive bond-breaking up to depth d=3 plus exhaustive hydrogen rearrangements; more exotic fragmentation schemes are ignored, which may limit recall.
- The GNNs consider atoms only and take no bond information beyond the enumeration heuristic, so positional/isomeric effects on fragment probabilities are not captured.
- The probability-weighted sum is an approximate spectrum synthesis — the paper's model is explicitly 'Rapid Approximate Subset-Based', and it matches the limiting experiment-to-experiment noise (~1.2%) rather than individual experimental spectra exactly.
- Practical build fragility: the numba and tinygraph dependencies require numpy<=1.21; a mismatch raises 'Numba needs NumPy 1.21 or less' or a tinygraph ValueError about binary incompatibility.

## Evidence

- [methods] Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect.: "Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect."
- [methods] Weight each formula/subset’s mass spectrum according to the model’s output probability and sum all of the observed mass spectra together: "We then weight each formula/subset’s mass spectrum according to the model’s output probability and sum all of the observed mass spectra together"
- [abstract] Both models can train on and predict spectral data at arbitrary resolution.: "Both models can train on and predict spectral data at arbitrary resolution"
- [readme] rassp/msutil — Fast code for computing mass spectra and molecule subsets: "`msutil`: Fast code for computing mass spectra and molecule subsets"
- [readme] README numpy pinning instruction — numpy must be 1.21 for numba/tinygraph compatibility: "    - `pip install numpy==1.21`"
- [readme] README spectrum schema — a spectrum is a list of (mass, intensity) 2-tuples: "- `spect`: List[Tuple[Float, Float]]
  - A spectrum is represented as a list of 2-tuple (mass, intensity) pairs."
- [readme] README forward evaluation uses normalize_pred: True, confirming unit-L2 normalization of predicted spectra: "'normalize_pred': True,"
- [intro] Abstract top-1/10 accuracy metrics to benchmark decoded spectra: FormulaNet 92.9% SDP, SubsetNet 91.8% (intro): [evidence span withheld — anchor status: not_found]
- [other] README limits: molecules ≤ 48 atoms (FormulaNet) / 64 (SubsetNet); only HCONFSPCl atoms are supported by the pipeline: "You can run inference on mols <= 48 atoms (for FormulaNet) and <= 64 atoms (for SubsetNet)."
