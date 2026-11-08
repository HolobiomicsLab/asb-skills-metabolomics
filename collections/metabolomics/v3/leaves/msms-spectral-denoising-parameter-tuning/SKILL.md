---
name: msms-spectral-denoising-parameter-tuning
description: Use when MS/MS query spectra were acquired on a non-Orbitrap high-resolution
  analyzer and the NIST23-derived electronic denoising threshold and mass tolerances
  must be re-derived and validated before applying Spectral Denoising.
when_to_use_negative:
- Query spectra were acquired on an Orbitrap instrument, where the existing NIST23-calibrated
  parameters (identical-intensity cutoff >4, 10-mDa threshold) are already valid
- Spectra are low-resolution or lack accurate precursor masses, since formula denoising
  requires chemically plausible subformula losses within a tight precursor mass window
- The goal is plain spectral cleaning without validation — parameter retuning is only
  justified when the analyzer-specific threshold must be re-derived and benchmarked
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: Applies electronic denoising (removing ions with identical intensities greater
    than 4, or the retuned analyzer-specific cutoff) followed by formula denoising
    (removing ions without a chemically plausible subformula loss); both parameters
    are retuned here
  repo: https://github.com/FanzhouKong/spectral_denoising
- name: Denoising Search
  role: Runs the denoise-and-match pipeline over all molecular formulas within the
    retuned precursor mass accuracy window, returning raw and denoised entropy similarities
    for candidate precursors
  repo: https://github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:148e52ffcaac3590b56cc09a0a86c8ba732f655e16995baeaba38d4d713455d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FanzhouKong/spectral_denoising
  tools:
  - Denoising Search
  - Spectral Denoising
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# msms-spectral-denoising-parameter-tuning

## Summary

Retunes and validates Spectral Denoising parameters (electronic-noise threshold and formula-denoising mass tolerance) when adapting the Orbitrap-optimized denoising method to a different high-resolution mass analyzer, using entropy similarity against reference spectra as the acceptance metric.

## When to use

Use when MS/MS query spectra were acquired on a non-Orbitrap high-resolution analyzer and the NIST23-derived electronic denoising threshold and mass tolerances must be re-derived and validated before applying Spectral Denoising.

## When NOT to use

- Query spectra were acquired on an Orbitrap instrument, where the existing NIST23-calibrated parameters (identical-intensity cutoff >4, 10-mDa threshold) are already valid
- Spectra are low-resolution or lack accurate precursor masses, since formula denoising requires chemically plausible subformula losses within a tight precursor mass window
- The goal is plain spectral cleaning without validation — parameter retuning is only justified when the analyzer-specific threshold must be re-derived and benchmarked

## Inputs

- Non-Orbitrap high-resolution MS/MS query spectra in .msp format
- Reference spectral library in .msp format (MassBank.us, GNPS, or package sample_data/reference_library.msp)
- Molecular formula or SMILES and adduct information for query compounds
- Precursor m/z values for each query spectrum

## Outputs

- Retuned electronic-denoising threshold and formula-denoising mass tolerance for the non-Orbitrap analyzer
- Denoised peak arrays (numpy [m/z, intensity] float32 arrays)
- denoising_eval_nonorbitrap.csv with per-spectrum pre- vs post-denoising entropy similarity values

## How to apply

Load the non-Orbitrap MS/MS query spectra and reference library with spectral_denoising.file_io.read_msp (sample_data MSP files or MassBank.us/GNPS spectra as the evaluation set). Re-derive the electronic-noise cutoff by empirically counting ions with identical intensities per spectrum in the new corpus and setting the threshold at the same rarity level (<0.05%) that produced the Orbitrap rule of removing ions with identical intensities greater than 4; that rule came from 230,000 NIST23 Orbitrap spectra where >99.5% of ions had fewer than four neighbors within ±0.1% relative intensity. Retune the formula-denoising mass tolerance for the analyzer's mass accuracy, keeping it within the 10-mDa precursor mass accuracy threshold used by Denoising Search. Run sd.spectral_denoising (electronic then formula denoising) and sd.denoising_search with the retuned profile, score pre- vs post-denoising spectra against references with spectral_operations.entropy_similairty supplying the precursor m/z so the precursor region is excluded, and accept the profile only if denoised similarity approaches the NIST23 Orbitrap benchmark of average similarity >0.99 and clearly exceeds the noisy baseline.

## Related tools

- **Spectral Denoising** (Applies electronic denoising (removing ions with identical intensities greater than 4, or the retuned analyzer-specific cutoff) followed by formula denoising (removing ions without a chemically plausible subformula loss); both parameters are retuned here) — https://github.com/FanzhouKong/spectral_denoising
- **Denoising Search** (Runs the denoise-and-match pipeline over all molecular formulas within the retuned precursor mass accuracy window, returning raw and denoised entropy similarities for candidate precursors) — https://github.com/FanzhouKong/spectral_denoising

## Examples

```
import spectral_denoising as sd; query = sd.read_msp('nonorbitrap_query.msp'); ref = sd.read_msp('sample_data/reference_library.msp'); result = sd.denoising_search(query.iloc[0]['peaks'], query.iloc[0]['precursor_mz'], ref)
```

## Evaluation signals

- Denoised non-Orbitrap spectra reach entropy similarity approaching the Orbitrap NIST23 benchmark of average similarity >0.99
- Post-denoising entropy similarity clearly exceeds the noisy-spectrum baseline similarity computed with entropy_similairty
- The empirical identical-intensity ion count distribution in the non-Orbitrap corpus places the retuned cutoff at the <0.05% rarity level, mirroring the NIST23 calibration
- Retuned formula-denoising mass tolerance stays within the maximum ±10 mDa precursor search window
- Wrapped in main() when using batch mode, since sd.spectral_denoising_batch and sd.denoising_search_batch use multiprocessing

## Limitations

- The electronic-noise threshold of fewer than four ions within ±0.1% relative intensity is instrument-specific, calibrated from 230,000 NIST23 Orbitrap spectra, so it cannot be transferred to other analyzers without re-derivation
- Chemical plausibility checks are limited to the Seven Golden Rules algorithm without molecular connectivity, so not all chemical noise ions are removed
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- The package requires Python 3.8–3.12 (RDKit has no compatible distribution for 3.13)

## Evidence

- [results] Electronic-noise threshold is Orbitrap-specific: "For more than 99.5% of all ions per spectrum, fewer than four ions were found within relative intensities of ±0.1%"
- [discussion] Parameters need retuning on other analyzers: "While our method is optimized for Orbitrap instruments, adapting it for other mass analyzers will require retuning and validating the parameters to ensure optimal performance."
- [methods] electronic_denoising removes ions with identical intensities greater than 4: "the ``electronic_denoising`` function removes ions with identical intensities greater than 4."
- [results] Benchmark similarity for denoised spectra: "The average similarity of the selected denoised NIST23 spectra was greater than 0.99"
- [results] Denoising Search uses a 10-mDa mass accuracy threshold: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [readme] Batch mode requires main() wrapper: [evidence span withheld — anchor status: external_doc]
