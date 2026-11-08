---
name: noise-ion-statistics
description: 'Use when calibrating an electronic (''grass'') noise filter for high-resolution
  MS/MS data: you need an empirical, instrument-specific intensity-coincidence threshold
  rather than a fixed base-peak-height cutoff.'
when_to_use_negative:
- When denoising spectra from non-Orbitrap mass analyzers — the article notes adapting
  the method to other instruments requires retuning and validating the parameters
- When the goal is removing chemical noise ions (subformula losses from the precursor),
  which requires formula_denoising rather than this intensity-coincidence statistic
- When input spectra are already denoised or are low-entropy reference spectra where
  the coincidence statistic is not the limiting factor
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: Provides the read_msp reader and electronic_denoising function whose threshold
    is calibrated by this empirical statistic
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: NumPy
  role: Array arithmetic for computing per-ion counts of near-identical (±0.1%) relative-intensity
    neighbors
- name: pandas
  role: Tabulates and aggregates the coincidence-count distribution across all spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:82487a4762775378d34ee79f0b3a39fd58d94d9a9a6200cd5ad4b2eb95dbf52f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - NumPy
  - Spectral Denoising
  - pandas
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# noise-ion-statistics

## Summary

Empirically derives the electronic-noise threshold for Orbitrap MS/MS denoising by counting, per ion, how many other ions in the same spectrum fall within ±0.1% relative intensity, across all 230,000 NIST23 Orbitrap spectra. The resulting statistic (>99.5% of ions have fewer than four such intensity-coincident neighbors) sets the cutoff used by the spectral_denoising electronic_denoising step.

## When to use

Use when calibrating an electronic ('grass') noise filter for high-resolution MS/MS data: you need an empirical, instrument-specific intensity-coincidence threshold rather than a fixed base-peak-height cutoff.

## When NOT to use

- When denoising spectra from non-Orbitrap mass analyzers — the article notes adapting the method to other instruments requires retuning and validating the parameters
- When the goal is removing chemical noise ions (subformula losses from the precursor), which requires formula_denoising rather than this intensity-coincidence statistic
- When input spectra are already denoised or are low-entropy reference spectra where the coincidence statistic is not the limiting factor

## Inputs

- MSP file of reference MS/MS spectra (e.g. 230,000 NIST23 Orbitrap spectra)
- Peak lists (mass, intensity arrays) per spectrum, sorted with zero-intensity ions removed
- Relative-intensity coincidence window parameter (±0.1%) and neighbor-count threshold (4)

## Outputs

- Per-spectrum and global summary table of intensity-coincidence count distributions
- Empirical electronic-noise threshold (fewer than four ions within ±0.1% relative intensity)
- Global fraction of ions below the threshold (expected >0.995 for NIST23 Orbitrap)

## How to apply

Load all reference spectra (e.g. the 230,000 NIST23 Orbitrap MS/MS spectra) with the spectral_denoising package's file_io.read_msp reader, applying basic cleaning (sorted peaks, zero-intensity ions removed via spectral_operations.remove_zero_ions). For every ion in each spectrum, compute the number of other ions whose relative intensity lies within ±0.1% (near-identical intensity neighbors), then tally per spectrum and globally the fraction of ions with fewer than four such neighbors. Aggregate the coincidence-count distribution into a summary table; the reported NIST23 result is that >99.5% of ions per spectrum have fewer than four intensity-coincident neighbors, with identical-intensity counts greater than 4 occurring in <0.05% of cases. Use 4 as the cutoff: electronic_denoising removes ions with identical intensities greater than 4, independent of relative intensity. Validate by confirming the recomputed global fraction exceeds 0.995.

## Related tools

- **Spectral Denoising** (Provides the read_msp reader and electronic_denoising function whose threshold is calibrated by this empirical statistic) — https://github.com/FanzhouKong/spectral_denoising
- **NumPy** (Array arithmetic for computing per-ion counts of near-identical (±0.1%) relative-intensity neighbors)
- **pandas** (Tabulates and aggregates the coincidence-count distribution across all spectra)

## Examples

```
import spectral_denoising as sd; query_data = sd.read_msp('nist23_orbitrap.msp'); denoised = sd.spectral_denoising_batch(query_data['peaks'], query_data['smiles'], query_data['adduct'])
```

## Evaluation signals

- Recomputed global fraction of ions with fewer than four ±0.1% intensity-coincident neighbors exceeds 0.995 across the 230,000 NIST23 Orbitrap spectra
- Fraction of ions with identical-intensity counts greater than 4 is below 0.05%, matching the reported NIST23 finding
- Per-spectrum peak lists are sorted and contain no zero-intensity ions after read_msp cleaning with remove_zero_ions
- Applying the derived threshold via electronic_denoising removes ions with identical intensities greater than 4 regardless of their relative intensity

## Limitations

- The ±0.1%/4-ion threshold is empirically derived from Orbitrap data; adapting it to other mass analyzers requires retuning and validating the parameters
- Electronic denoising alone does not remove chemical noise; chemical plausibility checks (Seven Golden Rules, without LEWIS and SENIOR checks or molecular connectivity) may leave some noise ions unremoved
- Reference spectra with inherently low spectral entropy remain vulnerable to noise ions, potentially leading to false negatives

## Evidence

- [results] Threshold derivation from NIST23 empirical distribution: "For more than 99.5% of all ions per spectrum, fewer than four ions were found within relative intensities of ±0.1%"
- [methods] Threshold application rule: "the ``electronic_denoising`` function removes ions with identical intensities greater than 4."
- [results] Threshold use as automatic electronic noise filter: "These data served as a valid threshold to automatically identify electronic noise ions in experimental MS/MS spectra and remove these ions, independent of the relative intensity"
- [methods] Basic cleaning prerequisite: "``spectral_operations.remove_zero_ions`` will remove ions with zero intensity from the query spectrum."
- [discussion] Instrument-specific tuning caveat: "While our method is optimized for Orbitrap instruments, adapting it for other mass analyzers will require retuning and validating the parameters to ensure optimal performance."
- [readme] Reader and batch entry point: "query_data = sd.read_msp('sample_data/noisy_spectra.msp')"
