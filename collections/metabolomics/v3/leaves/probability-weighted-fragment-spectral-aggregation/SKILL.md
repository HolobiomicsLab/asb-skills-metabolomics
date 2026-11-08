---
name: probability-weighted-fragment-spectral-aggregation
description: Use when you have a per-fragment probability distribution (atom subsets
  or chemical subformulae) with exact atomic-composition masses and need to render
  it as a summed mass spectrum at a specified bin resolution.
when_to_use_negative:
- You only need a per-fragment spectrum and not the final summed model prediction
  — stop at the per-fragment observed-spectrum step instead of aggregating.
- The fragment distribution carries no probability weights or the weights do not sum
  to 1; the aggregation relies on the model output probabilities from FormulaNet/SubsetNet.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RASSP (rassp-public)
  role: Provides msutil code for fast computation of mass spectra and molecule subsets,
    plus pre-trained FormulaNet/SubsetNet models whose output probability distributions
    feed this aggregation step.
  repo: https://github.com/thejonaslab/rassp-public
- name: numpy
  role: Used to bin fragment spectra into integer-Dalton or 0.10 Da bins, apply probability
    weighting, and sum the weighted spectra numerically.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:518812c7a49b091ed31b6b6b91d43ba9b278ecae5728c6fee385e21cc1ef22dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - RASSP (rassp-public)
  - numpy
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Probability-weighted fragment spectral aggregation

## Summary

Converts a model's predicted probability distribution over molecular fragments into a final mass spectrum by computing an isotope- and mass-defect-aware observed spectrum per fragment, weighting it by the fragment probability, and summing at the desired integer or sub-Dalton bin resolution.

## When to use

Use when you have a per-fragment probability distribution (atom subsets or chemical subformulae) with exact atomic-composition masses and need to render it as a summed mass spectrum at a specified bin resolution.

## When NOT to use

- You only need a per-fragment spectrum and not the final summed model prediction — stop at the per-fragment observed-spectrum step instead of aggregating.
- The fragment distribution carries no probability weights or the weights do not sum to 1; the aggregation relies on the model output probabilities from FormulaNet/SubsetNet.

## Inputs

- Enumerated fragment distribution (e.g. CSV where each fragment/atom subset or chemical subformula carries its model probability and exact atomic-composition masses)
- Target output bin resolution (e.g. 1 Da with integer-centered unit-width bins, or 0.10 Da high-resolution)
- Rounding rule for exact masses (round to integer Dalton for 1 Da output; no rounding for sub-Dalton)

## Outputs

- Final summed mass spectrum as (mass, intensity) pairs, e.g. tolulene_spectrum_1da.csv for 1 Da resolution
- High-resolution variant, e.g. toluene_spectrum_01da.csv (mass, intensity) at 0.10 Da bins
- Validation checks: non-negative intensities, unit-L2-normalizable intensity vector, probability-weighted total intensity equal to sum of fragment probabilities

## How to apply

Load the enumerated fragment distribution so that each fragment (atom subset or chemical subformula) carries its model probability and exact atomic-composition masses. For each fragment, compute the observed spectrum by summing atomic masses including isotopic variability at natural abundance and mass defect, then round to integer Dalton for 1 Da output bins (bins centered on integer values with unit widths) or keep exact masses for finer bins (e.g. 0.10 Da). Weight each fragment's spectrum by its model output probability and sum all the probability-weighted spectra to produce the final spectrum; this is exactly how RASSP converts FormulaNet/SubsetNet outputs into spectra. Write the result as (mass, intensity) pairs and check that the summed intensities are non-negative, unit-L2-normalizable, and that the total probability-weighted intensity equals the sum of fragment probabilities (mass conservation).

## Related tools

- **RASSP (rassp-public)** (Provides msutil code for fast computation of mass spectra and molecule subsets, plus pre-trained FormulaNet/SubsetNet models whose output probability distributions feed this aggregation step.) — https://github.com/thejonaslab/rassp-public
- **numpy** (Used to bin fragment spectra into integer-Dalton or 0.10 Da bins, apply probability weighting, and sum the weighted spectra numerically.)

## Examples

```
python rassp/run_rassp.py  # write smiles/inchi strings to sample_data/in.txt; spectra are stored in sample_data/out.txt
```

## Evaluation signals

- At both 1 Da and 0.10 Da resolutions the probability-weighted total spectral intensity equals the sum of fragment probabilities (mass conservation).
- All output intensities are non-negative.
- The final intensity vector is unit-L2-normalizable.
- Output CSV contains (mass, intensity) pairs with masses matching the chosen bin centers (integers with unit widths, or 0.10 Da grid).
- No fragment probability is lost: each fragment's spectrum appears once in the sum, weighted by its model output probability.

## Limitations

- Exotic fragmentation schemes are ignored, so their inclusion could potentially improve recall — the enumeration feeding this aggregation may miss rare fragments.
- The graph neural networks used upstream only consider the atoms and do not take into account any information about the bonds, so fragment probabilities may not capture bond-dependent behavior.
- Masses above the modeled atom limits (≤48 atoms for FormulaNet, ≤64 atoms for SubsetNet via the web API) cannot be produced.
- Mass-defect and isotopic variability are modeled, but the binning resolution (integer Da vs 0.10 Da) caps the detail of the final spectrum.

## Evidence

- [methods] Observation model: isotope-aware mass spectrum computation per formula/subset: "Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect."
- [methods] Probability-weighted summation of fragment spectra: "We then weight each formula/subset’s mass spectrum according to the model’s output probability and sum all of the observed mass spectra together"
- [intro] RASSP predicts probability distributions over reduced representations of molecular fragments: "rapid approximate subset-based spectra prediction” (RASSP) predicts probability distributions over reduced representations of molecular fragments—atom subsets (vertex"
- [readme] rassp msutil provides fast code for computing mass spectra: "`msutil`: Fast code for computing mass spectra and molecule subsets"
- [discussion] Weighted spectra summation limitation: exotic fragmentation schemes ignored: "However, there are more exotic fragmentation schemes that we have ignored, and their inclusion could potentially improve the recall"
