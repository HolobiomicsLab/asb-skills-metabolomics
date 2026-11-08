---
name: msms-intensity-coincidence-analysis
description: 'Use when calibrating an electronic-noise removal threshold for MS/MS
  spectra: you need to empirically justify how many ions per spectrum can share near-identical
  relative intensities (±0.1%) before denoising experimental Orbitrap data.'
when_to_use_negative:
- The input spectra are not from Orbitrap-type high-resolution instruments — the article
  notes adapting the method to other mass analyzers requires retuning and validating
  the parameters.
- The goal is chemical-noise removal (subformula-loss plausibility) rather than electronic
  ('grass') noise identification — use formula_denoising instead.
- The input is already a denoised spectrum or a search result table; this skill is
  a threshold-derivation/calibration step, not an annotation workflow.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: Package whose file_io.read_msp reader loads the NIST23 MSP spectra and whose
    electronic_denoising function consumes the derived threshold to remove ions with
    identical intensities greater than 4
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: NumPy
  role: Holds per-spectrum peak arrays ([mz, intensity] float32) on which the ±0.1%
    relative-intensity coincidence counts are computed
- name: pandas
  role: Tabulates per-spectrum and global aggregation of the intensity-coincidence
    counts across the 230,000 spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:18ae19e4eeb028fb5c202c6a04d991aa5c1a0be2c8eb144cc5d527bebaf59a39
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# msms-intensity-coincidence-analysis

## Summary

Empirical analysis that probes a large reference spectral library (230,000 NIST23 Orbitrap MS/MS spectra) to count near-identical-intensity ions per spectrum, establishing the ±0.1% relative-intensity coincidence threshold used by the Spectral Denoising algorithm to identify and remove electronic ('grass') noise ions.

## When to use

Use when calibrating an electronic-noise removal threshold for MS/MS spectra: you need to empirically justify how many ions per spectrum can share near-identical relative intensities (±0.1%) before denoising experimental Orbitrap data.

## When NOT to use

- The input spectra are not from Orbitrap-type high-resolution instruments — the article notes adapting the method to other mass analyzers requires retuning and validating the parameters.
- The goal is chemical-noise removal (subformula-loss plausibility) rather than electronic ('grass') noise identification — use formula_denoising instead.
- The input is already a denoised spectrum or a search result table; this skill is a threshold-derivation/calibration step, not an annotation workflow.

## Inputs

- 230,000 NIST23 Orbitrap MS/MS spectra in MSP format (NIST23 library)
- Per-spectrum peak lists (mz/intensity arrays, e.g. numpy float32 [mz, intensity] arrays as parsed by file_io.read_msp)
- Coincidence window parameter: ±0.1% relative intensity
- Coincidence count threshold: 4 identical-intensity neighbors

## Outputs

- Per-spectrum fraction of ions with fewer than four ±0.1%-intensity-coincident neighbors
- Global (library-wide) aggregated coincidence-count distribution summary table (pandas)
- Validated empirical threshold (>0.995 global fraction) for downstream electronic_denoising

## How to apply

Load all 230,000 NIST23 Orbitrap MS/MS spectra with the spectral_denoising package's file_io.read_msp reader, applying basic cleaning (sorted peaks, zero-intensity ions removed via spectral_operations.remove_zero_ions). For every ion in each spectrum, compute the number of other ions in the same spectrum whose relative intensity lies within ±0.1% (near-identical intensity neighbors). Tally per spectrum and globally the fraction of ions having fewer than four such intensity-coincident neighbors, aggregating the coincidence-count distribution into a summary table with NumPy/pandas. The empirical result — more than 99.5% of ions per spectrum have fewer than four ions within ±0.1% relative intensity — becomes the electronic_denoising rule that removes ions with identical intensities greater than 4, independent of relative intensity. This threshold is instrument- and library-specific: it was derived for Orbitrap data and would need retuning and validation for other mass analyzers.

## Related tools

- **Spectral Denoising** (Package whose file_io.read_msp reader loads the NIST23 MSP spectra and whose electronic_denoising function consumes the derived threshold to remove ions with identical intensities greater than 4) — https://github.com/FanzhouKong/spectral_denoising
- **NumPy** (Holds per-spectrum peak arrays ([mz, intensity] float32) on which the ±0.1% relative-intensity coincidence counts are computed)
- **pandas** (Tabulates per-spectrum and global aggregation of the intensity-coincidence counts across the 230,000 spectra)

## Examples

```
import spectral_denoising as sd; spectra = sd.read_msp('NIST23_orbitrap.msp')
```

## Evaluation signals

- The recomputed global fraction of ions with fewer than four ±0.1%-relative-intensity neighbors exceeds 0.995 across the 230,000 NIST23 Orbitrap spectra.
- The fraction of spectra containing identical-intensity counts greater than 4 is below 0.05%, reproducing the reported NIST23 finding.
- Per-spectrum peak lists are sorted by m/z and contain no zero-intensity ions after basic cleaning (remove_zero_ions applied).
- Applying the derived threshold via electronic_denoising removes electronic noise independent of relative intensity, e.g. robust even when noise ions exceed true fragment intensity by >150-fold.

## Limitations

- The ±0.1% / fewer-than-four threshold was empirically derived on Orbitrap NIST23 data; adapting to other mass analyzers requires retuning and validating the parameters.
- Identical-intensity coincidences greater than 4 are 'extremely unlikely' but not impossible, so the threshold can in principle misclassify a rare genuine ion as electronic noise.
- Denoising electronic noise alone does not address chemical noise or the known limitations of MS/MS-only annotation (isomeric compounds, in-source fragments, low-entropy reference spectra).

## Evidence

- [results] For more than 99.5% of all ions per NIST23 spectrum, fewer than four ions were found within relative intensities of ±0.1%: "For more than 99.5% of all ions per spectrum, fewer than four ions were found within relative intensities of ±0.1%"
- [results] Threshold used independent of relative intensity: "These data served as a valid threshold to automatically identify electronic noise ions in experimental MS/MS spectra and remove these ions, independent of the relative intensity"
- [methods] Empirical basis on NIST23: "According to empiracally tested on NIST23 database, in a given spectrum, the number of ions with identical intensities more than 4 is extremely unlikely"
- [methods] electronic_denoising implementation of the threshold: "the ``electronic_denoising`` function removes ions with identical intensities greater than 4."
- [readme] Reader and basic cleaning of input spectra: "``spectral_operations.remove_zero_ions`` will remove ions with zero intensity from the query spectrum."
