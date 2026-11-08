---
name: chemical-noise-sampling-from-formula-db
description: Use when benchmarking an MS/MS denoising or spectral-matching algorithm's
  robustness by deliberately contaminating clean spectra with chemically plausible
  synthetic noise ions whose intensity can far exceed true fragment ions.
when_to_use_negative:
- The input spectra are already experimental/noisy field data — this skill only manufactures
  synthetic noise; use the actual spectral_denoising or Denoising Search pipeline
  instead
- The mass analyzer is not an Orbitrap instrument — the article notes adapting the
  method to other mass analyzers requires retuning and validating parameters, so noise-generation
  thresholds may not transfer
- No precursor molecular formula or SMILES is available, since formula_denoising and
  chemically plausible noise sampling both depend on precursor formula information
tools:
- name: Spectral Denoising
  role: Provides sd.noise.generate_chemical_noise and sd.noise.add_noise to build
    formula_db-sampled chemical noise, plus sd.spectral_denoising to test denoising
    recovery of the contaminated spectra
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: ms_entropy
  role: Underlies sd.entropy_similairty, used to quantify similarity of contaminated
    and denoised spectra against ground-truth originals with the precursor region
    removed via the pmz argument
- name: numpy
  role: Holds the peaks arrays (m/z, intensity pairs as float32) that noise ions are
    merged into
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:e2aaeb7dae6e69bd3381f290e1be678f416730764b825eba2f6457e43683cf95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - Spectral Denoising
  - ms_entropy
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chemical-noise-sampling-from-formula-db

## Summary

Generates synthetic chemical noise for MS/MS robustness testing by drawing m/z values randomly from a formula_db reference (m/z range 50 to precursor m/z) with Poisson-distributed intensities, then merging the noise ions into clean query spectra via the spectral_denoising package.

## When to use

Use when benchmarking an MS/MS denoising or spectral-matching algorithm's robustness by deliberately contaminating clean spectra with chemically plausible synthetic noise ions whose intensity can far exceed true fragment ions.

## When NOT to use

- The input spectra are already experimental/noisy field data — this skill only manufactures synthetic noise; use the actual spectral_denoising or Denoising Search pipeline instead
- The mass analyzer is not an Orbitrap instrument — the article notes adapting the method to other mass analyzers requires retuning and validating parameters, so noise-generation thresholds may not transfer
- No precursor molecular formula or SMILES is available, since formula_denoising and chemically plausible noise sampling both depend on precursor formula information

## Inputs

- Clean query MS/MS spectra (.msp file, e.g. sample_data/query_spectra.msp) with peaks, smiles/formula, adduct and precursor_mz columns
- formula_db reference table of molecular formulas sorted by mass (Google Drive file distributed with the package)
- Precursor m/z and adduct information per query spectrum (e.g. [M+Na]+)

## Outputs

- Contaminated spectra: original peaks arrays merged with synthetic chemical noise ions (Poisson intensities, lambda=50) and optionally electronic noise (lambda=5)
- Entropy similarity scores of contaminated vs. ground-truth spectra (expected ~0.33–0.47 in the package examples)
- Denoised spectra and their entropy similarity to ground truth (expected ~0.99–1.00) for robustness validation

## How to apply

Load clean query spectra with sd.read_msp (e.g. sample_data/query_spectra.msp) and the formula_db reference used for noise generation. Generate synthetic chemical noise with sd.noise.generate_chemical_noise, which draws m/z values randomly from the formula_db over the range 50 to the precursor m/z and intensities from a Poisson distribution at high lambda (lambda=50); merge the noise into each true spectrum with sd.noise.add_noise. Optionally layer electronic 'grass' noise via sd.noise.generate_noise (uniform m/z over 50 to pmz, Poisson lambda=5) to test combined contamination. Run sd.spectral_denoising on each contaminated spectrum — it applies electronic_denoising (removes ions with identical intensities greater than 4) followed by formula_denoising (keeps ions explainable by a chemically plausible subformula loss validated with Seven Golden Rules checks, ignoring LEWIS and SENIOR). Score both contaminated and denoised spectra against the ground-truth original with sd.entropy_similairty, removing the precursor region via the pmz argument. The noise design is deliberately adversarial: noise ions are chemically plausible (sampled from real formulas) and can exceed true fragment-ion intensity by >150-fold.

## Related tools

- **Spectral Denoising** (Provides sd.noise.generate_chemical_noise and sd.noise.add_noise to build formula_db-sampled chemical noise, plus sd.spectral_denoising to test denoising recovery of the contaminated spectra) — https://github.com/FanzhouKong/spectral_denoising
- **ms_entropy** (Underlies sd.entropy_similairty, used to quantify similarity of contaminated and denoised spectra against ground-truth originals with the precursor region removed via the pmz argument)
- **numpy** (Holds the peaks arrays (m/z, intensity pairs as float32) that noise ions are merged into)

## Examples

```
import spectral_denoising as sd; chem_noise = sd.noise.generate_chemical_noise(peaks, pmz, smiles, formula_db, lambda=50); peak_noisy = sd.noise.add_noise(peaks, chem_noise)
```

## Evaluation signals

- Contaminated spectra must score ~0.33–0.47 entropy similarity against ground-truth originals (confirming noise was actually added and is severe)
- After sd.spectral_denoising, denoised spectra must recover ~0.99–1.00 entropy similarity to the originals
- Robustness must hold even when noise-ion intensities exceed true fragment-ion intensities by >150-fold
- All generated noise m/z values must fall within the 50 to precursor-m/z window drawn from the formula_db
- Spectral entropy of contaminated spectra should be visibly elevated (e.g. S = 3.3 vs S = 1.9 for the denoised example in the paper)

## Limitations

- Not all added noise ions will be removed because chemical plausibility checks are limited to the Seven Golden Rules method without considering molecular connectivity
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- The noise model assumes Orbitrap-style behavior; parameters (Poisson lambda, m/z range, identical-intensity threshold of 4) may need retuning for other mass analyzers
- Package requires Python 3.8–3.12; RDKit has no compatible distribution for Python 3.13

## Evidence

- [abstract] Robust against varying levels of chemical and electronic noise even with >150-fold higher intensity of noise ions than true fragment ions: [evidence span withheld — anchor status: not_found]
- [methods] draws m/z values randomly from the formula_db (m/z range 50 to precursor m/z) and intensities from a Poisson distribution at high lambda (lambda=50): [evidence span withheld — anchor status: not_found]
- [methods] The formula_db can be found at: `Formula_db <https://drive.google.com/file/d/1pEXiGc5l0YjRGfCEZXW7-Wz6D1dOSBxA/view?usp=drive_link>`_. It is already sorted by mass.: "The formula_db can be found at: `Formula_db <https://drive.google.com/file/d/1pEXiGc5l0YjRGfCEZXW7-Wz6D1dOSBxA/view?usp=drive_link>`_. It is already sorted by mass."
- [results] The average similarity of the selected denoised NIST23 spectra was greater than 0.99: "The average similarity of the selected denoised NIST23 spectra was greater than 0.99"
- [readme] generate some noise ions and add it to the peaks: [evidence span withheld — anchor status: external_doc]
