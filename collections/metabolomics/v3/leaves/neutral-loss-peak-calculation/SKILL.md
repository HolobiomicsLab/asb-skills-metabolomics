---
name: neutral-loss-peak-calculation
description: Use when per-compound MS/MS spectra spanning multiple collision energies
  (from MSP-derived data) must be transformed into a single enhanced spectrum containing
  fragment ions plus neutral-loss peaks in binarized (0/1) form before CNN-based metabolite
  identification.
when_to_use_negative:
- Spectra are already merged and intensity-binarized — re-applying the collapse would
  discard real collision-energy-specific structure.
- Intensity quantitation is the goal — binarization to 0/1 deliberately discards relative
  intensity information to reduce overfitting.
- The peak list should feed tools that require raw intensity values (e.g. library
  dot-product scoring against intensity-weighted reference spectra) rather than ChemEmbed's
  binarized CNN input.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ChemEmbed
  role: Provides the data-processing module implementing merge_spectra_by_feature-style
    collapse, neutral-loss calculation, and intensity binarization
  repo: github.com/massspecdl/ChemEmbed
- name: pandas >= 1.0.0
  role: Loads MSP-derived spectra into structured DataFrames and writes the merged
    spectrum table
- name: numpy >= 1.18.0
  role: Numeric computation of precursor-minus-fragment neutral-loss m/z values and
    0/1 binarization of intensities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:9c8f06dea787d098d32ce3a8a21d4e3f6c3682daffb57c97699d702c5cfef088
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  tools:
  - ChemEmbed
  - numpy >=
  - numpy >= 1.18.0
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# neutral-loss-peak-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enriches merged MS/MS spectra with calculated neutral-loss peaks (precursor m/z minus fragment m/z) and binarizes intensities, producing the enhanced spectral input used by ChemEmbed for metabolite identification.

## When to use

Use when per-compound MS/MS spectra spanning multiple collision energies (from MSP-derived data) must be transformed into a single enhanced spectrum containing fragment ions plus neutral-loss peaks in binarized (0/1) form before CNN-based metabolite identification.

## When NOT to use

- Spectra are already merged and intensity-binarized — re-applying the collapse would discard real collision-energy-specific structure.
- Intensity quantitation is the goal — binarization to 0/1 deliberately discards relative intensity information to reduce overfitting.
- The peak list should feed tools that require raw intensity values (e.g. library dot-product scoring against intensity-weighted reference spectra) rather than ChemEmbed's binarized CNN input.

## Inputs

- MSP-derived per-compound MS/MS spectra spanning multiple collision energies (structured DataFrames)
- Precursor m/z for each compound/feature
- Configuration parameters: intensity_threshold, peak-binning tolerance (--tolerance), m/z resolution

## Outputs

- One merged, binarized spectrum per compound containing both fragment-ion peaks and calculated neutral-loss peaks
- Spectrum table file ready as CNN model input for ChemEmbed embedding prediction

## How to apply

Load the per-compound MS/MS spectra across multiple collision energies from MSP-derived data using pandas/numpy within the ChemEmbed data-processing module. Collapse every MS2 scan of a compound/feature into one representative spectrum via the merge_spectra_by_feature-style collapse. Compute neutral-loss peaks as (precursor m/z − fragment m/z) and append them to the merged spectrum so fragments and losses co-exist in one peak list. Binarize all peak intensities to presence/absence (0/1), respecting the configured intensity_threshold and the --tolerance peak-binning parameter for peak alignment (note --tolerance affects binning only, not candidate retrieval). Write one merged, binarized spectrum per compound to the spectrum table file used downstream for CNN embedding prediction.

## Related tools

- **ChemEmbed** (Provides the data-processing module implementing merge_spectra_by_feature-style collapse, neutral-loss calculation, and intensity binarization) — github.com/massspecdl/ChemEmbed
- **pandas >= 1.0.0** (Loads MSP-derived spectra into structured DataFrames and writes the merged spectrum table)
- **numpy >= 1.18.0** (Numeric computation of precursor-minus-fragment neutral-loss m/z values and 0/1 binarization of intensities)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Each compound/feature yields exactly one merged spectrum row after the collapse (merge_spectra_by_feature collapses every MS2 scan of a feature into a single spectrum).
- All peak entries in the merged spectrum are binarized, taking only values 0 or 1.
- The merged spectrum contains both fragment-ion peaks and computed neutral-loss peaks (precursor m/z minus fragment m/z).
- Neutral-loss and fragment m/z values are aligned within the configured --tolerance peak-binning window and respect intensity_threshold.
- Downstream CNN training/inference accepts the table without errors; one epoch trains ~17x faster (~5.5 min vs ~95 min) with merged spectra than with individual collision-energy spectra.

## Limitations

- The reference database used downstream covers 520,083 compounds, so compounds outside it cannot be identified regardless of spectral quality.
- Binarization discards relative intensity information; individual collision-energy spectra are higher in sparsity and underperform merged spectra for training.
- The legacy exact 3-decimal bucket filter (precursor_tolerance_ppm=0) failed whenever the fourth decimal digit was 5 or above — about half of all compounds — and should not be used for new work.
- MSP handling differs depending on whether input files include SMILES annotations (--input_file_type with/without_smiles).

## Evidence

- [discussion] Neutral losses appended to merged spectrum: [evidence span withheld — anchor status: not_found]
