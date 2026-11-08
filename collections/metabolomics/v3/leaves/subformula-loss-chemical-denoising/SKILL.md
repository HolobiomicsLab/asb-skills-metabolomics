---
name: subformula-loss-chemical-denoising
description: Use when MS/MS spectra contain chemical noise ions — e.g. co-isolated
  contaminants or artificially generated chemical noise — and the precursor molecular
  formula and adduct are known, so fragments lacking chemically plausible subformula
  losses can be removed before spectral matching.
when_to_use_negative:
- The precursor molecular formula/adduct is unknown and no reference-library hit can
  supply it — use denoising_search, which pseudo-denoises against all molecular formulas
  within the precursor mass accuracy window, instead
- Input is a feature/peak table or chromatographic data rather than MS/MS peak arrays
  (m/z, intensity) from .msp files or packed numpy arrays
- Data were acquired on non-Orbitrap mass analyzers, where the parameters must be
  retuned and validated before use
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3521
tools:
- name: Spectral Denoising
  role: provides sd.spectral_denoising and its formula_denoising step implementing
    the subformula-loss chemical denoising
  repo: https://github.com/FanzhouKong/spectral_denoising.git
- name: RDKit
  role: cheminformatics dependency of the package for formula handling; requires Python
    3.8–3.12
- name: ms_entropy
  role: entropy-similarity scoring to quantify recovery against ground-truth spectra
    after denoising
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:40f3c3fa619455b645568e8be34afaef5d88289522654eeac834194ff10c8d2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising.git
  tools:
  - RDKit
  - Spectral Denoising
  - ms_entropy
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# subformula-loss-chemical-denoising

## Summary

Chemical denoising of MS/MS spectra as implemented in the spectral_denoising package (formula_denoising): each fragment ion's exact mass must be explainable as a chemically plausible subformula loss from the precursor molecular ion, with candidate losses validated by the Seven Golden Rules algorithm (LEWIS and SENIOR checks ignored). It removes chemical noise ions that electronic denoising cannot touch, enabling high-confidence compound annotation.

## When to use

Use when MS/MS spectra contain chemical noise ions — e.g. co-isolated contaminants or artificially generated chemical noise — and the precursor molecular formula and adduct are known, so fragments lacking chemically plausible subformula losses can be removed before spectral matching.

## When NOT to use

- The precursor molecular formula/adduct is unknown and no reference-library hit can supply it — use denoising_search, which pseudo-denoises against all molecular formulas within the precursor mass accuracy window, instead
- Input is a feature/peak table or chromatographic data rather than MS/MS peak arrays (m/z, intensity) from .msp files or packed numpy arrays
- Data were acquired on non-Orbitrap mass analyzers, where the parameters must be retuned and validated before use

## Inputs

- MS/MS query spectra as (m/z, intensity) peak arrays (numpy float32) or a DataFrame loaded from a .msp file via sd.read_msp with peaks/precursor_mz/smiles/formula/adduct columns
- Precursor molecular formula (or SMILES string) of the parent compound, from a reference-library top hit or computed
- Adduct string (e.g. '[M+H]+', '[M+Na]+'); precursor m/z via calculate_precursormz
- formula_db: mass-sorted reference formula database used to draw chemical-noise m/z values in robustness benchmarks

## Outputs

- Denoised peak array (or list of denoised spectra from spectral_denoising_batch) retaining only ions explainable by chemically plausible subformula losses from the precursor
- Entropy similarity scores (sd.entropy_similairty with pmz) of contaminated vs denoised spectra against ground-truth originals

## How to apply

Obtain the precursor molecular formula from the reference spectral library top hit or compute it with cheminformatic tools; the package accepts either a SMILES or a formula column (adduct, e.g. '[M+Na]+', gives the precursor m/z via calculate_precursormz). Run formula_denoising (executed automatically inside sd.spectral_denoising after electronic_denoising, which first removes ions with identical intensities >4): retain only fragment ions whose exact mass can be associated with a chemically plausible subformula loss from the parent molecular ion. Validate each relative-loss subformula with the Seven Golden Rules algorithm, discarding chemically impossible losses such as CH12, and account for possible [M+N2+H2O] molecule reactions for substituted aromatic compounds. To benchmark robustness, contaminate true spectra with sd.noise.generate_chemical_noise (m/z drawn from a mass-sorted formula_db between 50 and the precursor m/z, Poisson lambda=50 intensities) and score recovery with sd.entropy_similairty using the pmz argument to exclude the precursor region. Expect denoised-vs-original entropy similarity of ~0.99–1.00 (contaminated ~0.33–0.47); in the NIST23 Orbitrap benchmark the average similarity of selected denoised spectra exceeded 0.99, and recovery holds even at >150-fold higher noise-ion intensity than true fragment ions.

## Related tools

- **Spectral Denoising** (provides sd.spectral_denoising and its formula_denoising step implementing the subformula-loss chemical denoising) — https://github.com/FanzhouKong/spectral_denoising.git
- **RDKit** (cheminformatics dependency of the package for formula handling; requires Python 3.8–3.12)
- **ms_entropy** (entropy-similarity scoring to quantify recovery against ground-truth spectra after denoising)

## Examples

```
import spectral_denoising as sd; peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)
```

## Evaluation signals

- Entropy similarity of the denoised spectrum vs its ground-truth original must reach ~0.99–1.00 (contaminated spectra score ~0.33–0.47) with sd.entropy_similairty and the precursor region removed via pmz
- In the NIST23 benchmark, average similarity of selected denoised spectra must exceed 0.99
- Every retained fragment must map to a subformula loss passing the Seven Golden Rules; impossible losses such as CH12 must be discarded while LEWIS and SENIOR checks are ignored
- Recovery must persist when chemical-noise ion intensities exceed true fragment-ion intensities by >150-fold (Poisson lambda=50 noise generation)
- Spectral entropy should decrease after denoising (published example: contaminated spectrum S = 3.3 vs denoised S = 1.9)

## Limitations

- Not all added chemical noise ions are removed because chemical plausibility checks are limited to the Seven Golden Rules method without considering molecular connectivity
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- The method is optimized for Orbitrap instruments; adapting it to other mass analyzers requires retuning and validating parameters
- Python 3.8–3.12 is required (RDKit has no Python 3.13 distribution), and batch mode must be wrapped in a main() function when run from a terminal because it uses multiprocessing

## Evidence

- [methods] The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion.: "The ``formula_denoising`` function removes chemical noise ions in MS/MS spectra by evaluating if it could be formed from a chemically plausible subformula loss to the precursor ion."
- [results] validated using the Seven Golden Rules algorithm to discard chemically impossible losses (for example, CH12)32 while ignoring the LEWIS and SENIOR checks: "validated using the Seven Golden Rules algorithm to discard chemically impossible losses (for example, CH12)32 while ignoring the LEWIS and SENIOR checks"
- [results] Spectral Denoising accounts for possible [M+N2+H2O] molecule reactions when calculating the relative loss of fragment ions for substituted aromatic compounds: "Spectral Denoising accounts for possible [M+N2+H2O] molecule reactions when calculating the relative loss of fragment ions for substituted aromatic compounds"
- [results] The average similarity of the selected denoised NIST23 spectra was greater than 0.99: "The average similarity of the selected denoised NIST23 spectra was greater than 0.99"
- [abstract] even with >150-fold higher intensity of noise ions than true fragment ions: "even with >150-fold higher intensity of noise ions than true fragment ions"
- [discussion] our chemical plausibility checks were limited to the algorithms embedded in the Seven Golden Rules method32, without considering: "our chemical plausibility checks were limited to the algorithms embedded in the Seven Golden Rules method32, without considering"
- [readme] peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct): "peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)"
