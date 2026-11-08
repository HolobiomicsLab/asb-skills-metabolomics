---
name: isotope-distribution-prediction
description: Use when the input is a molecular formula (e.g., C3H8O2) and you need
  its monoisotopic exact mass, adduct formula/mz, or predicted M, M+1, M+2 isotope-peak
  masses and abundances for MS assignment.
when_to_use_negative:
- The input is already raw MS data (MZA/HDF5, mzML, vendor .raw/.d) needing extraction
  or conversion rather than formula-based isotope prediction
- A full theoretical isotope distribution beyond M/M+1/M+2 is required — predict_m_m1_m2
  covers only the first three peaks
- The formulas are unknown and must first be determined by peak finding or calibration;
  use mzapy's peaks/calibration modules instead
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzapy
  role: Provides the isotopes module (monoiso_mass, ms_adduct_formula, ms_adduct_mz,
    predict_m_m1_m2, OrderedMolecularFormula) used to compute exact masses and isotope
    distributions from molecular formulas
  repo: https://github.com/PNNL-m-q/mzapy
- name: Python
  role: Implementation language in which the isotope calculations are executed; mzapy
    is installable from PyPI via pip
  repo: https://github.com/PNNL-m-q/mzapy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:74c599ff3a78524e3538de03b8da795482a31a3987308080af4f347a1d573c3d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# isotope-distribution-prediction

## Summary

Computes monoisotopic exact masses and theoretical M/M+1/M+2 isotopologue masses and relative abundances from molecular formulas using mzapy.isotopes, whose element exact-mass table derives from unimod.org.

## When to use

Use when the input is a molecular formula (e.g., C3H8O2) and you need its monoisotopic exact mass, adduct formula/mz, or predicted M, M+1, M+2 isotope-peak masses and abundances for MS assignment.

## When NOT to use

- The input is already raw MS data (MZA/HDF5, mzML, vendor .raw/.d) needing extraction or conversion rather than formula-based isotope prediction
- A full theoretical isotope distribution beyond M/M+1/M+2 is required — predict_m_m1_m2 covers only the first three peaks
- The formulas are unknown and must first be determined by peak finding or calibration; use mzapy's peaks/calibration modules instead

## Inputs

- Molecular formulas as element/count strings (e.g., C3H8O2) read from an input file
- Optional adduct specification for computing adduct formulas and m/z values

## Outputs

- Monoisotopic exact mass per molecular formula
- Predicted M, M+1, and M+2 isotopologue masses with relative abundances per formula
- Adduct formulas and adduct m/z values (when adducts are specified)
- JSON file (e.g., isotope_profile.json) collecting computed masses and isotope abundances per formula

## How to apply

Read the target element/count formulas (e.g., C3H8O2) from the input file and instantiate each with mzapy.isotopes.OrderedMolecularFormula, which subclasses MolecularFormula and orders elements by increasing mass. Compute each formula's monoisotopic exact mass with mzapy.isotopes.monoiso_mass, which uses the _ELEMENT_MONOISO_MASS table of most-abundant-isotope exact masses (source: unimod.org masses). Where charge/adduct context is needed, derive the adduct formula with ms_adduct_formula and its m/z with ms_adduct_mz. Predict the M, M+1, and M+2 peak masses and relative abundances with mzapy.isotopes.predict_m_m1_m2, then write masses and abundances per formula into an output such as isotope_profile.json. Rationale: this supplies theoretical isotope patterns against which observed MS peaks can be matched, complementing mzapy's peak-finding utilities (find_peaks_1d_gauss, calc_peak_area).

## Related tools

- **mzapy** (Provides the isotopes module (monoiso_mass, ms_adduct_formula, ms_adduct_mz, predict_m_m1_m2, OrderedMolecularFormula) used to compute exact masses and isotope distributions from molecular formulas) — https://github.com/PNNL-m-q/mzapy
- **Python** (Implementation language in which the isotope calculations are executed; mzapy is installable from PyPI via pip) — https://github.com/PNNL-m-q/mzapy

## Examples

```
from mzapy import isotopes; mf = isotopes.OrderedMolecularFormula({'C': 3, 'H': 8, 'O': 2}); m0 = isotopes.monoiso_mass(mf); m, m1, m2, ab = isotopes.predict_m_m1_m2(mf)
```

## Evaluation signals

- Each computed monoisotopic mass equals the sum of the per-element most-abundant-isotope exact masses from the mzapy.isotopes element table within small numerical tolerance
- Predicted M/M+1/M+2 relative abundances are consistent with published isotope tables for C/H/O/N-containing formulas
- Every formula instantiates successfully as an OrderedMolecularFormula with elements ordered by increasing mass
- The output isotope_profile.json contains mass and abundance entries for every input formula with no missing records
- Adduct m/z values from ms_adduct_mz are consistent with the adduct formulas produced by ms_adduct_formula and the charge state

## Limitations

- predict_m_m1_m2 predicts only the M, M+1, and M+2 peaks, not a complete isotope envelope
- Monoisotopic masses come from the _ELEMENT_MONOISO_MASS table of most-abundant isotopes sourced from unimod.org; results are bounded by that table's values
- No changelog was found in the article's discussion, so version-specific behavior changes cannot be traced from the provided context

## Evidence

- [methods] mzapy's isotopes module exposes functions such as monoiso_mass, ms_adduct_formula, ms_adduct_mz, and predict_m_m1_m2 that compute monoisotopic mass, adduct formulas/masses, and predicted isotopologue masses for a molecular formula.: [evidence span withheld — anchor status: not_found]
