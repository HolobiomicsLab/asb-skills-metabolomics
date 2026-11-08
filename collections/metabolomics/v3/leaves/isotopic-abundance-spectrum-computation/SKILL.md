---
name: isotopic-abundance-spectrum-computation
description: Use when converting a predicted probability distribution over molecular
  fragments (subformulae or atom subsets with known exact atomic-composition masses)
  into a discrete mass spectrum at a chosen integer-Dalton or sub-Dalton resolution.
when_to_use_negative:
- Input is already a measured spectrum (e.g. a NIST 2017 Main Library entry); this
  skill constructs a predicted spectrum from fragment probabilities, not observed
  data.
- The molecule contains atoms outside HCONFSPCl or exceeds the enumeration limits
  (≤48 atoms for FormulaNet, ≤4096 unique fragment formulae), so the fragment distribution
  was never produced.
- You need fragmentation pathways or bond-cleavage mechanisms rather than an isotope-resolved
  observation model — RASSP ignores exotic fragmentation schemes and bond information
  beyond atoms.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: RASSP (rassp-public)
  role: Provides the msutil fast code for computing mass spectra and molecule subsets
    that implements the isotope-aware observation model and probability-weighted spectral
    summation
  repo: https://github.com/thejonaslab/rassp-public
- name: numpy
  role: Array arithmetic for binning isotope-resolved exact masses into integer- or
    0.1-Da bins and summing probability-weighted fragment spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:d02701df803b04e4d963c9a752c1ce0878eeda532eb127f45541dd6229f05a08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - RASSP (rassp-public)
  - numpy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# isotopic-abundance-spectrum-computation

## Summary

Computes the exact observed mass spectrum of a molecular fragment formula or atom subset by convolving natural-abundance isotope distributions with per-element mass defects, then bins the result to a target resolution (1 Da or 0.1 Da) and weights it by the model's fragment probability before summing into the final predicted EI–MS spectrum.

## When to use

Use when converting a predicted probability distribution over molecular fragments (subformulae or atom subsets with known exact atomic-composition masses) into a discrete mass spectrum at a chosen integer-Dalton or sub-Dalton resolution.

## When NOT to use

- Input is already a measured spectrum (e.g. a NIST 2017 Main Library entry); this skill constructs a predicted spectrum from fragment probabilities, not observed data.
- The molecule contains atoms outside HCONFSPCl or exceeds the enumeration limits (≤48 atoms for FormulaNet, ≤4096 unique fragment formulae), so the fragment distribution was never produced.
- You need fragmentation pathways or bond-cleavage mechanisms rather than an isotope-resolved observation model — RASSP ignores exotic fragmentation schemes and bond information beyond atoms.

## Inputs

- Enumerated fragment distribution: list of fragments (atom subsets or chemical subformulae) each with model output probability and exact atomic-composition masses (e.g. CSV of toluene fragments)
- Molecular formula or atom subset from RASSP FormulaNet (≤48 atoms, ≤4096 unique fragment formulae) or SubsetNet (≤64 atoms)
- Target spectral resolution: 1 Da (integer-Dalton bins centered on integers, unit widths) or 0.10 Da (unrounded exact masses)

## Outputs

- Probability-weighted summed mass spectrum as (mass, intensity) pairs at 1 Da resolution, e.g. tolulene_spectrum_1da.csv
- Probability-weighted summed mass spectrum as (mass, intensity) pairs at 0.10 Da resolution, e.g. toluene_spectrum_01da.csv
- Per-fragment observed spectra (intermediate), each binned at the target resolution before weighting

## How to apply

For each enumerated fragment (from RASSP's subformula enumeration via setwise Cartesian products, or bond-breaking subsets to depth d=3 plus exhaustive hydrogen rearrangements), compute the exact observed spectrum: sum atomic masses incorporating isotopic variability at natural abundance and mass defect to obtain the isotope-resolved m/z distribution. Round masses to integer Daltons and accumulate intensity into unit-width bins centered on integers for 1 Da output; keep unrounded masses and use 0.10 Da-wide bins for the high-resolution variant. Multiply each fragment's spectrum by the model's output probability for that fragment (FormulaNet subformula probability or SubsetNet subset probability from mean-pooled atom embeddings) and sum all probability-weighted fragment spectra into the final predicted spectrum. Rationale: the paper's observation model states that a given formula's spectrum can be exactly calculated once isotopic natural abundance and mass defect are accounted for, and the final spectrum is the probability-weighted sum over all fragments. Write outputs as (mass, intensity) pairs, e.g. tolulene_spectrum_1da.csv and toluene_spectrum_01da.csv, then verify mass-conservation invariants before use in downstream Stein-dot-product or library-lookup evaluation.

## Related tools

- **RASSP (rassp-public)** (Provides the msutil fast code for computing mass spectra and molecule subsets that implements the isotope-aware observation model and probability-weighted spectral summation) — https://github.com/thejonaslab/rassp-public
- **numpy** (Array arithmetic for binning isotope-resolved exact masses into integer- or 0.1-Da bins and summing probability-weighted fragment spectra)

## Examples

```
conda activate rassp; cd rassp; python run_rassp.py  # reads sample_data/in.txt, writes spectra to sample_data/out.txt
```

## Evaluation signals

- Mass conservation: at both 1 Da and 0.1 Da resolutions, the probability-weighted total spectral intensity equals the sum of fragment probabilities.
- Intensity non-negativity: the summed spectrum contains only non-negative intensities and is unit-L2-normalizable.
- Binning check: 1 Da output bins are centered on integer values with unit widths; 0.1 Da output uses unrounded exact masses with 0.10 Da bins.
- Probability weighting check: each fragment's spectrum contributes proportionally to its model output probability (FormulaNet/SubsetNet), with contributions summed over all fragments.
- Schema check: output CSVs contain mass and intensity columns, e.g. tolulene_spectrum_1da.csv and toluene_spectrum_01da.csv.

## Limitations

- The observation model accounts only for natural-abundance isotope variability and mass defect; more exotic fragmentation schemes are ignored, and their inclusion could potentially improve recall.
- The graph neural networks used only consider the atoms and do not take into account any information about the bonds, so isotope/fragment predictions reflect atom composition rather than connectivity.
- Numpy versioning is fragile: Numba requires NumPy ≤1.21 and the Tinygraph binary is incompatible with NumPy ≥1.20 API changes (pin numpy==1.21 after uninstalling).
- Applicability limits: FormulaNet handles molecules with ≤48 atoms and ≤4096 unique fragment formulae; SubsetNet handles ≤64 atoms.

## Evidence

- [methods] Observation model with isotopes and mass defect: "Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect."
- [methods] Probability-weighted summation produces the final spectrum: "We then weight each formula/subset’s mass spectrum according to the model’s output probability and sum all of the observed mass spectra together"
- [methods] Probability-weighted mechanism summary: [evidence span withheld — anchor status: not_found]
- [methods] Enumeration limits for applicability: "After filtering the dataset down to molecules containing only HCONFSPCl atoms, with total atoms ≤ 48, number of unique fragment formulae ≤ 4096"
- [readme] Local install and numpy pinning: "pip uninstall numpy
    - `pip install numpy==1.21`"
- [readme] msutil implements spectra and subsets: "msutil: Fast code for computing mass spectra and molecule subsets"
- [discussion] Bonds are not modeled: "The graph neural networks we use only consider the atoms and do not take into account any information about the bonds"
