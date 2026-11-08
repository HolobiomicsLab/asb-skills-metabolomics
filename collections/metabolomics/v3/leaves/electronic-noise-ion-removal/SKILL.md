---
name: electronic-noise-ion-removal
description: Use when a raw MS/MS spectrum (e.g., from an Orbitrap instrument, loaded
  from .msp) contains co-occurring 'grass' noise ions with identical intensities that
  must be stripped before chemical denoising or spectral similarity matching.
when_to_use_negative:
- The noise ions are chemical noise with distinct intensities and chemically meaningful
  masses — these require formula_denoising against a precursor molecular formula,
  not the identical-intensity rule
- The data come from a non-Orbitrap mass analyzer — the paper states the method is
  optimized for Orbitrap instruments and adapting it requires retuning and validating
  parameters
- The spectrum is already clean and only precursor-region trimming is needed — use
  remove_precursor or remove_zero_ions instead of the noise-identity rule
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spectral Denoising
  role: Python package whose spectral_denoising.electronic_denoising function implements
    the >4 identical-intensity noise removal rule
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: numpy
  role: Holds the float32 peak array (m/z, intensity) that electronic_denoising operates
    on
- name: pandas
  role: Dataframe handling for .msp query spectra loaded by sd.read_msp (peaks, smiles,
    formula, adduct, precursor_mz columns)
- name: ms_entropy
  role: Provides spectral_entropy / entropy_similarity used to validate the denoised
    spectrum against a clean reference
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:b64dda4d2fc79b31b9719831ec76eff1fe0a7e6048f4c29c0120ad8b668aea7f
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
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# electronic-noise-ion-removal

## Summary

Removes electronic ('grass') noise ions from a raw MS/MS spectrum using the Spectral Denoising package's electronic_denoising filter, which deletes any group of more than four ions with near-identical intensities (within ±0.1% relative intensity), a threshold derived empirically from the NIST23 Orbitrap library.

## When to use

Use when a raw MS/MS spectrum (e.g., from an Orbitrap instrument, loaded from .msp) contains co-occurring 'grass' noise ions with identical intensities that must be stripped before chemical denoising or spectral similarity matching.

## When NOT to use

- The noise ions are chemical noise with distinct intensities and chemically meaningful masses — these require formula_denoising against a precursor molecular formula, not the identical-intensity rule
- The data come from a non-Orbitrap mass analyzer — the paper states the method is optimized for Orbitrap instruments and adapting it requires retuning and validating parameters
- The spectrum is already clean and only precursor-region trimming is needed — use remove_precursor or remove_zero_ions instead of the noise-identity rule

## Inputs

- Raw noisy MS/MS spectrum as a peak array (m/z, intensity columns), e.g., loaded from sample_data/noisy_spectra.msp via sd.read_msp
- Optional: spectra from a full .msp query file for batch processing (spectral_denoising_batch)

## Outputs

- Cleaned peak array (m/z and intensity columns) with electronic noise ions removed, e.g., saved as electronic_denoised_spectrum.csv
- Intermediate cleaned spectrum suitable as input to the downstream formula_denoising (chemical noise) step

## How to apply

Load the noisy spectrum with spectral_denoising.file_io.read_msp, which sorts peaks and drops zero-intensity ions. Apply sd.electronic_denoising to the peak array (m/z and intensity columns): this standalone first step flags any group of more than four ions sharing near-identical intensities as electronic noise and removes them, independent of absolute relative intensity — deliberately not a simple 1% base-peak cutoff like classic thresholding, DNL denoising, or MS Reduce. The rule is empirically grounded in NIST23: for more than 99.5% of ions per spectrum, fewer than four ions fall within ±0.1% relative intensity, so >4 identical-intensity ions is extremely unlikely (<0.05%) in validated spectra. Low-abundant true fragment ions are retained while co-occurring identical-intensity noise ions are eliminated. No precursor molecular-formula information is needed at this stage; formula_denoising runs afterwards if precursor/adduct information (e.g., SMILES, formula, adduct such as [M+Na]+) is available.

## Related tools

- **Spectral Denoising** (Python package whose spectral_denoising.electronic_denoising function implements the >4 identical-intensity noise removal rule) — https://github.com/FanzhouKong/spectral_denoising
- **numpy** (Holds the float32 peak array (m/z, intensity) that electronic_denoising operates on)
- **pandas** (Dataframe handling for .msp query spectra loaded by sd.read_msp (peaks, smiles, formula, adduct, precursor_mz columns))
- **ms_entropy** (Provides spectral_entropy / entropy_similarity used to validate the denoised spectrum against a clean reference)

## Examples

```
import spectral_denoising as sd; peak_denoised = sd.electronic_denoising(sd.read_msp('sample_data/noisy_spectra.msp').iloc[0]['peaks'])
```

## Evaluation signals

- Entropy similarity of the denoised spectrum against a clean reference at the precursor m/z recovers ~1.00, versus ~0.37 for the noisy spectrum (documented example)
- No more than four ions with near-identical intensities (within ±0.1% relative intensity) remain in the output spectrum
- Low-abundant true fragment ions present in the input are retained in the cleaned peak array, not collapsed by a 1% base-peak height cutoff
- Zero-intensity ions are absent from the output (read_msp/remove_zero_ions drops them before filtering)
- In noise-contamination benchmarks, denoised spectra score ~0.99–1.00 entropy similarity to ground truth versus ~0.33–0.47 contaminated, even when noise intensities exceed true fragment ions by >150-fold

## Limitations

- The >4 identical-intensity threshold is empirically derived from NIST23 Orbitrap spectra and is optimized for Orbitrap instruments; other mass analyzers require retuning and validating parameters
- Electronic denoising alone does not remove chemical noise ions with distinct intensities; chemical plausibility checks are limited to Seven Golden Rules without molecular connectivity, so not all chemical noise is removed
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives

## Evidence

- [methods] electronic_denoising threshold rule: "the ``electronic_denoising`` function removes ions with identical intensities greater than 4."
- [results] NIST23 empirical basis: "For more than 99.5% of all ions per spectrum, fewer than four ions were found within relative intensities of ±0.1%"
- [results] intensity-independent removal: "These data served as a valid threshold to automatically identify electronic noise ions in experimental MS/MS spectra and remove these ions, independent of the relative intensity"
- [readme] package workflow on sample data: "peak_with_noise= sd.read_msp('sample_data/noisy_spectra.msp').iloc[0]['peaks']
peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)"
- [abstract] robustness finding: "even with >150-fold higher intensity of noise ions than true fragment ions"
