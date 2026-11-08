---
name: molecular-fragment-annotation
description: Use when a model outputs a probability distribution over molecular fragments
  (atom subsets or chemical subformulae) with known exact masses and you need to convert
  those into a final mass spectrum at 1 Da or 0.1 Da resolution.
when_to_use_negative:
- The target spectrum is already directly predicted peak-by-peak from a fingerprint
  or molecular structure (e.g. NEIMS-style direct regression) rather than via a fragment
  probability distribution
- The molecule contains atoms outside HCONFSPCl, has > 48 atoms (FormulaNet) or >
  64 atoms (SubsetNet), or would yield > 4096 unique fragment formulae
- Collision-induced-dissociation MS2 or sub-Dalton high-resolution regimes where exotic
  fragmentation schemes ignored by this enumeration would dominate
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: RASSP
  role: Provides the fragment enumeration (getSubformulae, bond-breaking to d=3, hydrogen
    rearrangements), isotope-aware exact mass/spectrum computation, and probability-weighted
    spectral summation used to annotate and bin fragments
  repo: thejonaslab/rassp-public
- name: numpy
  role: Array arithmetic for binning probability-weighted fragment spectra into integer-Dalton
    and 0.1 Da bins and summing them into the final spectrum
- name: FormulaNet
  role: Produces the probability distribution over chemical subformulae that supplies
    fragment probabilities to be weighted into the spectrum
- name: SubsetNet
  role: Produces the probability distribution over vertex subsets of the molecule
    graph, giving fragment identities and probabilities for the spectral summation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:dca86410f0a31743d80e25880bfea5a2611bb1c20092596e71fcaf3985fa6ddc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - FormulaNet
  - RASSP
  - SubsetNet
  - numpy
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# molecular-fragment-annotation

## Summary

Annotates the fragments underlying a predicted or observed EI mass spectrum by enumerating a molecule's vertex (atom) subsets and chemical subformulae — via recursive bond-breaking and hydrogen rearrangements — then weighting each fragment's exactly computed isotope-aware spectrum by the model's predicted probability and summing them into a final spectrum.

## When to use

Use when a model outputs a probability distribution over molecular fragments (atom subsets or chemical subformulae) with known exact masses and you need to convert those into a final mass spectrum at 1 Da or 0.1 Da resolution.

## When NOT to use

- The target spectrum is already directly predicted peak-by-peak from a fingerprint or molecular structure (e.g. NEIMS-style direct regression) rather than via a fragment probability distribution
- The molecule contains atoms outside HCONFSPCl, has > 48 atoms (FormulaNet) or > 64 atoms (SubsetNet), or would yield > 4096 unique fragment formulae
- Collision-induced-dissociation MS2 or sub-Dalton high-resolution regimes where exotic fragmentation schemes ignored by this enumeration would dominate

## Inputs

- Enumerated fragment distribution CSV (fragment, model probability, exact atomic-composition masses) for the target molecule
- Molecular structure (SMILES/InChI) or molecular graph with per-atom features
- Target spectral resolution: 1 Da (integer Dalton rounding) or 0.10 Da (unrounded)

## Outputs

- Final predicted mass spectrum as (mass, intensity) pairs at 1 Da resolution (tolulene_spectrum_1da.csv)
- Final predicted mass spectrum as (mass, intensity) pairs at 0.1 Da resolution (toluene_spectrum_01da.csv)
- Intermediate per-fragment exact observed spectra weighted by model probability

## How to apply

Enumerate each molecule's possible fragments by recursively breaking bonds out to depth d=3, perform exhaustive hydrogen rearrangements, and (for FormulaNet) generate all chemical subformulae by recursively taking the setwise Cartesian product of per-element subformulae. For each fragment, compute the exact observed spectrum, taking into account isotopic variability at natural abundance and mass defect, then bin into integer-Dalton bins centered on integer values with unit widths (or 0.1 Da bins without rounding for high resolution). Weight each fragment's mass spectrum by the model's output probability and sum all observed mass spectra together to produce the final spectrum. Write the resulting (mass, intensity) lists to CSV, e.g. tolulene_spectrum_1da.csv and toluene_spectrum_01da.csv. Validation: intensities must be non-negative and unit-L2-normalizable, and the probability-weighted total intensity must equal the sum of fragment probabilities (mass conservation).

## Related tools

- **RASSP** (Provides the fragment enumeration (getSubformulae, bond-breaking to d=3, hydrogen rearrangements), isotope-aware exact mass/spectrum computation, and probability-weighted spectral summation used to annotate and bin fragments) — thejonaslab/rassp-public
- **numpy** (Array arithmetic for binning probability-weighted fragment spectra into integer-Dalton and 0.1 Da bins and summing them into the final spectrum)
- **FormulaNet** (Produces the probability distribution over chemical subformulae that supplies fragment probabilities to be weighted into the spectrum)
- **SubsetNet** (Produces the probability distribution over vertex subsets of the molecule graph, giving fragment identities and probabilities for the spectral summation)

## Examples

```
python -m pip install -e . && cd rassp && conda activate rassp && python run_rassp.py  # with sample_data/in.txt holding SMILES/InChI strings
```

## Evaluation signals

- Summed spectrum intensities are all non-negative and unit-L2-normalizable
- At both 1 Da and 0.1 Da resolutions, probability-weighted total spectral intensity equals the sum of fragment probabilities (mass conservation)
- Bin centers sit exactly on integer Dalton values with unit widths for the 1 Da output (no fractional bin centers)
- Output CSVs (tolulene_spectrum_1da.csv, toluene_spectrum_01da.csv) have exactly (mass, intensity) columns with no missing fragment contributions
- Predicted spectrum achieves Stein dot product consistent with reported forward-accuracy benchmarks (e.g. 92.9% for FormulaNet) when compared against the NIST 2017 reference

## Limitations

- More exotic fragmentation schemes are ignored and their inclusion could potentially improve recall
- Graph neural networks used for scoring consider only atoms and do not take into account any information about the bonds
- Numpy versioning conflicts: numba requires numpy <= 1.21 and tinygraph breaks at numpy >= 1.20.0, so pin numpy==1.21 after install

## Evidence

- [methods] Probability-weighted spectral summation: "We then weight each formula/subset’s mass spectrum according to the model’s output probability and sum all of the observed mass spectra together"
- [methods] Isotope-aware exact spectrum per fragment: "Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect."
- [methods] Bond-breaking enumeration depth: "we adopt a heuristic bond-breaking approach where we begin with an initial molecule and recursively break all possible bonds out to a particular depth"
- [methods] Subformula Cartesian product enumeration: "we can iteratively generate all subformulae by recursively taking the setwise Cartesian product of the possible subformulae for a single element"
- [methods] Molecule-size filtering thresholds: "After filtering the dataset down to molecules containing only HCONFSPCl atoms, with total atoms ≤ 48, number of unique fragment formulae ≤ 4096"
- [discussion] Exotic fragmentation schemes omitted: "However, there are more exotic fragmentation schemes that we have ignored, and their inclusion could potentially improve the recall"
- [discussion] GNN ignores bond information: "The graph neural networks we use only consider the atoms and do not take into account any information about the bonds"
- [readme] Numpy versioning fix: "pip uninstall numpy
 pip install numpy==1.21"
- [readme] Inference size caps per model: "You can run inference on mols <= 48 atoms (for FormulaNet) and <= 64 atoms (for SubsetNet)."
