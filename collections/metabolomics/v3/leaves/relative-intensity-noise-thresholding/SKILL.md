---
name: relative-intensity-noise-thresholding
description: Use when a raw MS/MS spectrum (e.g. from an Orbitrap instrument, .msp
  format) contains co-occurring 'grass' electronic noise ions that must be removed
  before precursor-formula-dependent chemical denoising or spectral matching.
when_to_use_negative:
- The noise is chemical rather than electronic — fragments whose masses cannot be
  explained by chemically plausible subformula losses from the precursor require the
  formula_denoising step and precursor molecular-formula information, not this intensity-based
  rule
- The spectrum was acquired on a non-Orbitrap mass analyzer — the threshold was empirically
  derived from NIST23 Orbitrap MS/MS spectra and adapting it to other mass analyzers
  requires retuning and validating parameters
- The noise pattern is not characterized by more than four co-occurring near-identical-intensity
  ions (e.g. sparse isolated noise peaks), which this rule will not flag
tools:
- name: Spectral Denoising
  role: Python package providing the electronic_denoising function that removes ions
    with identical intensities greater than 4, plus spectral_operations utilities
    (remove_zero_ions) and entropy similarity metrics for validation
  repo: github.com/FanzhouKong/spectral_denoising
- name: numpy
  role: Holds the peak array (float32 m/z and intensity columns) that is passed to
    electronic_denoising
- name: pandas
  role: Handles the tabular spectrum data returned by sd.read_msp and the cleaned
    spectrum written to CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:231c1ddc605667a5794153751bf9a55da04ab63996ecd1cf005974d7cdb578f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spectral Denoising
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# relative-intensity-noise-thresholding

## Summary

An empirical, intensity-independent rule for removing electronic ('grass') noise ions from raw MS/MS spectra: any group of more than four ions with nearly identical relative intensities (within ±0.1%) is flagged as electronic noise and removed. This is the electronic_denoising step of the Spectral Denoising algorithm, derived by probing 230,000 Orbitrap MS/MS spectra in the NIST23 library.

## When to use

Use when a raw MS/MS spectrum (e.g. from an Orbitrap instrument, .msp format) contains co-occurring 'grass' electronic noise ions that must be removed before precursor-formula-dependent chemical denoising or spectral matching.

## When NOT to use

- The noise is chemical rather than electronic — fragments whose masses cannot be explained by chemically plausible subformula losses from the precursor require the formula_denoising step and precursor molecular-formula information, not this intensity-based rule
- The spectrum was acquired on a non-Orbitrap mass analyzer — the threshold was empirically derived from NIST23 Orbitrap MS/MS spectra and adapting it to other mass analyzers requires retuning and validating parameters
- The noise pattern is not characterized by more than four co-occurring near-identical-intensity ions (e.g. sparse isolated noise peaks), which this rule will not flag

## Inputs

- Raw MS/MS spectrum as a peak array (m/z and intensity columns), e.g. from sd.read_msp('sample_data/noisy_spectra.msp')
- Noisy spectra .msp file for batch processing

## Outputs

- Cleaned peak array (m/z, intensity) with electronic noise ions removed
- Intermediate denoised spectrum CSV (e.g. electronic_denoised_spectrum.csv) for downstream chemical denoising or spectral matching

## How to apply

Load the raw noisy spectrum with spectral_denoising.file_io.read_msp (e.g. sample_data/noisy_spectra.msp), which sorts peaks and drops zero-intensity ions. Apply sd.electronic_denoising to the peak array: the function removes ions whose identical intensities occur more than 4 times, because empirical analysis of the NIST23 database showed that in a given spectrum the number of ions with identical intensities more than 4 is extremely unlikely (<0.05%; more than 99.5% of ions per spectrum had fewer than four ions within ±0.1% relative intensities). Crucially, the rule is applied independent of the ions' relative intensity — it is NOT a 1% base-peak height cutoff like classic thresholding, DNL, or MS Reduce denoising. The output is a cleaned peak array (m/z and intensity columns) retaining low-abundant true fragments while eliminating identical-intensity noise ion groups; it operates as a standalone first step before any precursor molecular-formula information is needed. Validate by computing sd.entropy_similairty against a clean reference at the precursor m/z — the documented example recovers a similarity of 1.00 versus 0.37 for the noisy spectrum.

## Related tools

- **Spectral Denoising** (Python package providing the electronic_denoising function that removes ions with identical intensities greater than 4, plus spectral_operations utilities (remove_zero_ions) and entropy similarity metrics for validation) — github.com/FanzhouKong/spectral_denoising
- **numpy** (Holds the peak array (float32 m/z and intensity columns) that is passed to electronic_denoising)
- **pandas** (Handles the tabular spectrum data returned by sd.read_msp and the cleaned spectrum written to CSV)

## Examples

```
peaks = sd.read_msp('sample_data/noisy_spectra.msp').iloc[0]['peaks']; peak_denoised = sd.electronic_denoising(peaks)
```

## Evaluation signals

- After denoising, no more than four ions in the spectrum share nearly identical relative intensities (within ±0.1%); all groups of >4 identical-intensity ions have been removed
- entropy_similairty between the denoised spectrum and the clean reference at the precursor m/z reaches ~1.00, versus 0.37 for the noisy input spectrum
- Spectral entropy decreases from the contaminated spectrum (S = 3.3 in the paper's example) toward the denoised value (S = 1.9)
- Low-abundant true fragment ions are retained — the removed ions are identified solely by the >4 identical-intensity rule, not by a relative-intensity (e.g. 1% base-peak) cutoff

## Limitations

- The >4-identical-intensity threshold was empirically derived from ~230,000 Orbitrap MS/MS spectra of the NIST23 library and is instrument-specific; adapting to other mass analyzers requires retuning and validating the parameters
- This step only removes electronic noise; chemical noise ions require the formula_denoising step with precursor molecular-formula information
- The package requires Python between 3.8 and 3.12 (RDKit has no compatible distribution for Python 3.13)
- True fragment ions that coincidentally share identical intensities with more than four other ions could be incorrectly removed, since the rule ignores relative intensity

## Evidence

- [methods] NIST23 empirical threshold for identical-intensity ion groups: "According to empiracally tested on NIST23 database, in a given spectrum, the number of ions with identical intensities more than 4 is extremely unlikely"
- [results] 99.5% of ions have fewer than four near-identical-intensity neighbors: "For more than 99.5% of all ions per spectrum, fewer than four ions were found within relative intensities of ±0.1%"
- [methods] Electronic_denoising removes ions with identical intensities >4: "the ``electronic_denoising`` function removes ions with identical intensities greater than 4."
- [readme] Noise classification rationale from the package README: "Noise ions in MS/MS spectra are largely categorized as 1. electronic noises and 2. chemical noises."
- [readme] Validation via entropy similarity on the noisy-spectra example: "peak_denoised = sd.spectral_denoising(peak_with_noise, smiles, adduct)"
