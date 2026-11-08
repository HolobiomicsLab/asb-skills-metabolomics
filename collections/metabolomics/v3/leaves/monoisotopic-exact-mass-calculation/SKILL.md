---
name: monoisotopic-exact-mass-calculation
description: Use when you need theoretical monoisotopic masses or isotope distributions
  (M, M+1, M+2) for element-count molecular formulas (e.g., C3H8O2) in MS method development
  or adduct m/z prediction.
when_to_use_negative:
- Input is raw MS data (MZA/HDF5 spectra) needing extraction or peak finding rather
  than theoretical isotope calculations
- Formulas require full isotope envelopes beyond M+2 or isotope-resolved simulation
  of complex mixtures, since predict_m_m1_m2 covers only M, M+1, M+2
- The input already contains measured monoisotopic masses (e.g., PrecursorMonoisotopicMz
  from MZA metadata) that need no theoretical prediction
edam_operation: http://edamontology.org/operation_3441
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzapy
  role: Provides mzapy.isotopes functions (monoiso_mass, ms_adduct_formula, ms_adduct_mz,
    predict_m_m1_m2) and the OrderedMolecularFormula class used to compute monoisotopic
    masses and isotope distributions
  repo: https://github.com/PNNL-m-q/mzapy
- name: Python
  role: Runtime environment; mzapy is implemented in pure Python with minimal standardized
    dependencies
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:e827bed03ec746844a96dbb7dd3085e4b6b1e0689f01974e0b3e920dae131614
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  edam_operation: http://edamontology.org/operation_3441
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# monoisotopic-exact-mass-calculation

## Summary

Compute monoisotopic exact masses and M/M+1/M+2 isotopologue masses and abundances for molecular formulas using mzapy.isotopes, which bases its calculations on a most-abundant-isotope exact-mass element table sourced from unimod.org.

## When to use

Use when you need theoretical monoisotopic masses or isotope distributions (M, M+1, M+2) for element-count molecular formulas (e.g., C3H8O2) in MS method development or adduct m/z prediction.

## When NOT to use

- Input is raw MS data (MZA/HDF5 spectra) needing extraction or peak finding rather than theoretical isotope calculations
- Formulas require full isotope envelopes beyond M+2 or isotope-resolved simulation of complex mixtures, since predict_m_m1_m2 covers only M, M+1, M+2
- The input already contains measured monoisotopic masses (e.g., PrecursorMonoisotopicMz from MZA metadata) that need no theoretical prediction

## Inputs

- Element-count molecular formulas (e.g., C3H8O2) read from an input file
- Optional adduct specification for ms_adduct_formula / ms_adduct_mz

## Outputs

- Monoisotopic exact mass per molecular formula
- M, M+1, M+2 predicted peak masses with relative abundances
- isotope_profile.json recording computed masses and isotope abundances per formula

## How to apply

Read element-count molecular formulas from the input file (e.g., C3H8O2 as demonstrated in the mzapy.isotopes documentation) and instantiate each with mzapy.isotopes.OrderedMolecularFormula, a MolecularFormula subclass that orders elements by increasing mass. Compute the monoisotopic exact mass with mzapy.isotopes.monoiso_mass, which sums per-element most-abundant-isotope exact masses from the _ELEMENT_MONOISO_MASS table (source: unimod.org masses). For isotope distributions, call mzapy.isotopes.predict_m_m1_m2 to obtain the M, M+1, and M+2 peak masses with their relative abundances. If adduct m/z values are needed, use ms_adduct_formula to derive the adduct formula and ms_adduct_mz to compute its m/z. Write the computed masses and abundances per formula to an output such as isotope_profile.json.

## Related tools

- **mzapy** (Provides mzapy.isotopes functions (monoiso_mass, ms_adduct_formula, ms_adduct_mz, predict_m_m1_m2) and the OrderedMolecularFormula class used to compute monoisotopic masses and isotope distributions) — https://github.com/PNNL-m-q/mzapy
- **Python** (Runtime environment; mzapy is implemented in pure Python with minimal standardized dependencies)

## Examples

```
from mzapy.isotopes import OrderedMolecularFormula, monoiso_mass, predict_m_m1_m2; f = OrderedMolecularFormula({'C':3,'H':8,'O':2}); m = monoiso_mass(f); m, m1, m2, a1, a2 = predict_m_m1_m2(f)
```

## Evaluation signals

- Each computed monoisotopic mass equals the sum of the per-element most-abundant-isotope exact masses from the mzapy.isotopes element table (_ELEMENT_MONOISO_MASS) within small numerical tolerance
- M/M+1/M+2 relative abundances are consistent with published isotope tables for C/H/O/N-containing formulas
- OrderedMolecularFormula instantiation succeeds and orders elements by increasing mass
- isotope_profile.json contains an entry with masses and abundances for every input formula

## Limitations

- predict_m_m1_m2 covers only the M, M+1, and M+2 isotopologues, not full isotope envelopes
- Masses derive from a fixed element exact-mass table (_ELEMENT_MONOISO_MASS, source: unimod.org masses); no provision for custom isotope tables is described
- The README itself is truncated and does not document the isotopes module; usage details come from the article's methods listing and workflow

## Evidence
