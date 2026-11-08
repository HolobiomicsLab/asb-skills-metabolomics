---
name: synthetic-noise-spike-in-validation
description: Use when benchmarking a spectral denoising or noise-removal method and
  you need a quantitative recovery metric from spectra with known ground truth (clean
  reference plus controlled spiked noise).
when_to_use_negative:
- When no clean reference spectrum exists for the compound — similarity to ground
  truth cannot be computed and you must instead use Denoising Search against reference
  libraries
- When evaluating annotation performance on real experimental data (e.g., AD plasma
  samples) where ground-truth noise labels are unavailable; use Denoising Search metrics
  instead
- For instruments other than Orbitrap without retuning — the paper notes parameters
  are optimized for Orbitrap instruments and adaptation requires revalidation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: Python package whose spectral_denoising, electronic_denoising, and formula_denoising
    functions are the denoising pipeline being validated via noise spike-in
  repo: github.com/FanzhouKong/spectral_denoising
- name: numpy
  role: Builds and manipulates float32 peak arrays (m/z, intensity) used to construct
    noisy test spectra
- name: pandas
  role: Handles .msp query data loaded via sd.read_msp for batch denoising tests
- name: MS Reduce
  role: Alternative denoising method used as a benchmarking comparator in the spike-in
    evaluation
- name: DNL denoising
  role: Alternative dynamic-noise-level denoising method used as a benchmarking comparator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:c5e462dcaab1064d8ea767588019696c8f2fa0076f43e04b328a431164d94944
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNL denoising
  - MS Reduce
  - Spectral Denoising
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# synthetic-noise-spike-in-validation

## Summary

Validates an MS/MS denoising method by contaminating clean reference spectra with synthetic electronic/chemical noise ions, then quantifying recovery via spectral entropy similarity between the denoised output and the pristine reference.

## When to use

Use when benchmarking a spectral denoising or noise-removal method and you need a quantitative recovery metric from spectra with known ground truth (clean reference plus controlled spiked noise).

## When NOT to use

- When no clean reference spectrum exists for the compound — similarity to ground truth cannot be computed and you must instead use Denoising Search against reference libraries
- When evaluating annotation performance on real experimental data (e.g., AD plasma samples) where ground-truth noise labels are unavailable; use Denoising Search metrics instead
- For instruments other than Orbitrap without retuning — the paper notes parameters are optimized for Orbitrap instruments and adaptation requires revalidation

## Inputs

- Clean reference MS/MS peak arrays (m/z, intensity) from NIST23 or sample_data
- Precursor molecular information: SMILES string or molecular formula (e.g., 'C5H4N4O')
- Adduct type (e.g., '[M+Na]+') for precursor m/z calculation
- Synthetic noise ions: electronic noise (many ions with near-identical intensities) and chemical noise ions added to the clean spectrum

## Outputs

- Denoised peak arrays (m/z and intensity columns) after electronic and chemical denoising
- Entropy similarity scores between denoised spectra and clean references (e.g., 1.00 recovered vs 0.37 for noisy input)
- Spectral entropy / normalized entropy values before and after denoising (e.g., S = 1.9 denoised vs S = 3.3 contaminated)
- Benchmark comparison table against alternative denoising methods (1% base-peak cutoff, DNL, MS Reduce)

## How to apply

Start from clean reference spectra (e.g., NIST23 Orbitrap entries or the package's sample_data raw peaks arrays) and add synthetic noise ions — electronic 'grass' noise sharing near-identical intensities and chemical noise ions that are not plausible subformula losses of the precursor. Apply the denoising pipeline under test, here sd.spectral_denoising (electronic_denoising plus formula_denoising), passing the precursor SMILES/formula and adduct so chemical denoising can evaluate subformula losses. Score recovery with sd.entropy_similairty(peak_denoised, peak, pmz=pmz) between the denoised spectrum and the clean reference at the precursor m/z; a value of 1.00 indicates full recovery, while the contaminated input scores far lower (e.g., 0.37 in the documented example). Stress-test robustness by scaling noise-ion intensity — the paper reports the method remains robust even with >150-fold higher noise-ion intensity than true fragment ions — and benchmark against alternatives such as 1% base-peak thresholding, DNL denoising, and MS Reduce on a metabolite panel (240 tested metabolites).

## Related tools

- **Spectral Denoising** (Python package whose spectral_denoising, electronic_denoising, and formula_denoising functions are the denoising pipeline being validated via noise spike-in) — github.com/FanzhouKong/spectral_denoising
- **numpy** (Builds and manipulates float32 peak arrays (m/z, intensity) used to construct noisy test spectra)
- **pandas** (Handles .msp query data loaded via sd.read_msp for batch denoising tests)
- **MS Reduce** (Alternative denoising method used as a benchmarking comparator in the spike-in evaluation)
- **DNL denoising** (Alternative dynamic-noise-level denoising method used as a benchmarking comparator)

## Examples

```
import spectral_denoising as sd; peak_denoised = sd.spectral_denoising(sd.read_msp('sample_data/noisy_spectra.msp').iloc[0]['peaks'], 'O=c1nc[nH]c2nc[nH]c12', '[M+Na]+')
```

## Evaluation signals

- Entropy similarity between denoised spectrum and clean reference equals 1.00 (documented example recovers 1.00 vs 0.37 for the noisy spectrum)
- Average similarity across selected denoised NIST23 spectra is greater than 0.99
- Spectral entropy drops from contaminated levels (e.g., S = 3.3) toward clean-reference levels (e.g., S = 1.9)
- Recovery holds under stress: method stays robust even with >150-fold higher noise-ion intensity than true fragment ions
- Low-abundant true fragment ions are retained while spiked noise ions are removed, outperforming 1% base-peak cutoff, DNL, and MS Reduce on the 240-metabolite benchmark

## Limitations

- Chemical plausibility checks limited to Seven Golden Rules without molecular connectivity, so not all spiked chemical noise ions are removed
- Reference spectra with inherently low spectral entropy (<0.5, fewer than two fragment ions) are particularly vulnerable to noise ions, potentially causing false negatives
- Method optimized for Orbitrap instruments; applying the same spike-in thresholds to other mass analyzers requires retuning and revalidation
- Requires Python 3.8–3.12; RDKit has no compatible distribution for Python 3.13

## Evidence

- [abstract] Robust against varying levels of chemical and electronic noise even with >150-fold higher intensity of noise ions than true fragment ions: [evidence span withheld — anchor status: not_found]
- [results] Average similarity of denoised NIST23 spectra > 0.99: "The average similarity of the selected denoised NIST23 spectra was greater than 0.99"
- [readme] Entropy similarity between denoised and raw spectrum computed with pmz: [evidence span withheld — anchor status: external_doc]
- [methods] electronic_denoising removes ions with identical intensities greater than 4: "the ``electronic_denoising`` function removes ions with identical intensities greater than 4."
