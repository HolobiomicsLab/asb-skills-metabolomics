---
name: subformula-loss-mass-tolerance-retuning
description: Use when adapting the Spectral Denoising / Denoising Search pipeline
  to MS/MS data from a non-Orbitrap high-resolution analyzer, where the Orbitrap-calibrated
  electronic-noise rule (fewer than four ions at ±0.1% relative intensity) and mass
  tolerances may not transfer.
when_to_use_negative:
- Data were acquired on an Orbitrap instrument, where the NIST23-calibrated defaults
  (identical intensities > 4, 10 mDa threshold) already apply without retuning
- Spectra are low-quality references with spectral entropy <0.5 (fewer than two fragment
  ions), which are excluded from this kind of denoising evaluation
- The goal is distinguishing isomeric compounds or in-source fragments, which MS/MS
  spectral matching alone cannot resolve even after denoising
edam_topics:
- http://edamontology.org/topic_3521
tools:
- name: Spectral Denoising
  role: Performs electronic denoising (removes ions with identical intensities greater
    than 4) followed by formula denoising (removes ions lacking a chemically plausible
    subformula loss); both thresholds must be retuned for the non-Orbitrap analyzer
  repo: github.com/FanzhouKong/spectral_denoising
- name: Denoising Search
  role: Pseudo-denoises query spectra using molecular formulas within the predefined
    precursor mass accuracy (10 mDa) and returns precursor candidates with entropy
    similarities of raw and denoised spectra
  repo: github.com/FanzhouKong/spectral_denoising
- name: spectral_operations.entropy_similairty
  role: Scores denoised spectra against references with the precursor m/z supplied
    so the precursor region is excluded, providing pre- vs post-denoising validation
    metrics
  repo: github.com/FanzhouKong/spectral_denoising
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:c5725c67ea24245194bf6498ec3b60c35c5f4d8f5cc51a2fb3ba4ccc083e1378
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Denoising Search
  - Spectral Denoising
  - spectral_operations.entropy_similairty
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Subformula-loss mass tolerance retuning for non-Orbitrap analyzers

## Summary

Retunes and validates the Spectral Denoising parameters — the electronic-noise identical-intensity cutoff and the formula-denoising mass tolerance — that were empirically calibrated on Orbitrap NIST23 data before applying the method to a non-Orbitrap high-resolution mass analyzer.

## When to use

Use when adapting the Spectral Denoising / Denoising Search pipeline to MS/MS data from a non-Orbitrap high-resolution analyzer, where the Orbitrap-calibrated electronic-noise rule (fewer than four ions at ±0.1% relative intensity) and mass tolerances may not transfer.

## When NOT to use

- Data were acquired on an Orbitrap instrument, where the NIST23-calibrated defaults (identical intensities > 4, 10 mDa threshold) already apply without retuning
- Spectra are low-quality references with spectral entropy <0.5 (fewer than two fragment ions), which are excluded from this kind of denoising evaluation
- The goal is distinguishing isomeric compounds or in-source fragments, which MS/MS spectral matching alone cannot resolve even after denoising

## Inputs

- Non-Orbitrap high-resolution MS/MS query spectra in .msp format (query_spectra.msp-style file with peaks, precursor_mz, smiles/formula, adduct columns)
- Reference spectral library in .msp format (e.g. MassBank.us or GNPS spectra for the target analyzer)
- Molecular formula reference (smiles or formula column; formula_db sorted by mass for denoising search)

## Outputs

- Retuned parameter profile: analyzer-specific electronic-noise identical-intensity cutoff and formula-denoising mass tolerance (bounded by the ±10 mDa precursor search window)
- Per-spectrum pre- vs post-denoising entropy similarity scores (denoising_eval_nonorbitrap.csv-style table)

## How to apply

Load the non-Orbitrap query spectra and reference library with spectral_denoising.file_io.read_msp (MSP format, using the package's sample_data or MassBank.us/GNPS spectra). Re-derive the electronic denoising threshold by empirically counting identical-intensity ions per spectrum in the non-Orbitrap corpus, replacing the Orbitrap/NIST23-derived rule of removing ions with identical intensities greater than 4, and set the cutoff at the same <0.05% rarity level. Retune the formula denoising mass tolerance for the analyzer's mass accuracy, starting small and allowing it to grow only within the maximum ±10 mDa precursor search window. Run spectral_denoising (electronic then formula denoising) and denoising_search, scoring each denoised spectrum against its reference with spectral_operations.entropy_similairty with the precursor m/z supplied so the precursor region is excluded; accept the retuned profile if denoised entropy similarity approaches the Orbitrap benchmark of average similarity >0.99 (NIST23 evaluation) and clearly exceeds the noisy-spectrum baseline.

## Related tools

- **Spectral Denoising** (Performs electronic denoising (removes ions with identical intensities greater than 4) followed by formula denoising (removes ions lacking a chemically plausible subformula loss); both thresholds must be retuned for the non-Orbitrap analyzer) — github.com/FanzhouKong/spectral_denoising
- **Denoising Search** (Pseudo-denoises query spectra using molecular formulas within the predefined precursor mass accuracy (10 mDa) and returns precursor candidates with entropy similarities of raw and denoised spectra) — github.com/FanzhouKong/spectral_denoising
- **spectral_operations.entropy_similairty** (Scores denoised spectra against references with the precursor m/z supplied so the precursor region is excluded, providing pre- vs post-denoising validation metrics) — github.com/FanzhouKong/spectral_denoising

## Examples

```
import spectral_denoising as sd; query_spectra = sd.read_msp('sample_data/query_spectra.msp'); reference_library = sd.read_msp('sample_data/reference_library.msp'); result = sd.denoising_search(query_spectra.iloc[0]['peaks'], query_spectra.iloc[0]['precursor_mz'], reference_library)
```

## Evaluation signals

- Denoised spectra reach entropy similarity approaching the Orbitrap NIST23 benchmark of average similarity >0.99
- Post-denoising entropy similarity clearly exceeds the noisy-spectrum baseline for the same query-reference pairs
- Per-spectrum spectral entropy decreases after denoising (e.g. the paper's example dropped from S = 3.3 contaminated to S = 1.9 denoised)
- The retuned electronic cutoff preserves the original rarity criterion: fewer than four ions within ±0.1% relative intensities for >99.5% of ions per spectrum
- Retuned formula-denoising mass tolerance stays within the maximum ±10 mDa precursor search window

## Limitations

- The electronic-noise threshold is instrument-specific: it was calibrated from 230,000 Orbitrap NIST23 spectra, so it does not automatically transfer to other mass analyzers
- Chemical plausibility checks are limited to the Seven Golden Rules algorithm without considering molecular connectivity, so not all chemical noise ions are removed
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- Denoising Search must be supplemented with orthogonal measures (retention time matching, molecular cross-section comparisons, biological metadata screening); hard thresholding on spectral similarity alone is suboptimal

## Evidence

- [discussion] While our method is optimized for Orbitrap instruments, adapting it for other mass analyzers will require retuning and validating the parameters to ensure optimal performance.: "While our method is optimized for Orbitrap instruments, adapting it for other mass analyzers will require retuning and validating the parameters to ensure optimal performance."
- [methods] According to empiracally tested on NIST23 database, in a given spectrum, the number of ions with identical intensities more than 4 is extremely unlikely: "According to empiracally tested on NIST23 database, in a given spectrum, the number of ions with identical intensities more than 4 is extremely unlikely"
- [results] For practical reasons, a 10-mDa mass accuracy threshold was used: "For practical reasons, a 10-mDa mass accuracy threshold was used"
- [results] The chemical plausibility of relative loss subformulas was validated using the Seven Golden Rules algorithm to discard chemically impossible losses: "The chemical plausibility of relative loss subformulas was validated using the Seven Golden Rules algorithm to discard chemically impossible losses"
- [methods] the electronic_denoising`` function removes obvious electronic noise ions in MS/MS spectra: "The ``electronic_denoising`` function removes obvious electronic noise ions in MS/MS spectra"
