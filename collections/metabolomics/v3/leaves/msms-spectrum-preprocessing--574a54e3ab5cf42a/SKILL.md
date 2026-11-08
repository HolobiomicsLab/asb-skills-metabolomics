---
name: msms-spectrum-preprocessing
description: Use when input is a raw MSP file of MS/MS peak lists (e.g. Orbitrap or
  QTOF scans at several collision energies) that must be turned into fixed-width model
  inputs for embedding prediction or candidate matching.
when_to_use_negative:
- Input is already preprocessed model input (e.g. a binned spectrum array or embedding)
  — re-binning or re-merging would discard information.
- Only a single collision-energy spectrum exists per feature and no merging is intended;
  the preprocessing step still applies (thresholding/binning) but the merged-spectrum
  rationale does not.
- Candidate retrieval stage is the target — precursor tolerance (--precursor_tolerance_ppm),
  not --tolerance, governs filtering there; this skill does not change retrieval.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ChemEmbed
  role: Provides the preprocessing module that converts MSP files to DataFrames, merges
    spectra by feature, applies intensity thresholding and peak binning prior to CNN
    prediction
  repo: https://github.com/faizanurv/ChemEmbed
- name: pandas
  role: Structured DataFrame format for parsed MSP spectra (pandas >= 1.0.0)
- name: numpy
  role: Array operations on binned peak intensities (numpy >= 1.18.0)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:62edf77075b490dfb25eb09c1f3076f2d75ab4b1c915ef31edd22f832a5158c9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  repo_url: https://github.com/faizanurv/ChemEmbed
  tools:
  - ChemEmbed
  - Mol2vec
  - PyTorch
  - RDKit
  - numpy
  - numpy >=
  - pandas
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# MS/MS spectrum preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert raw MSP peak lists into model-ready spectra by merging scans across collision energies, binarizing intensities above a threshold, binning peaks at a fixed m/z resolution, and appending calculated neutral losses — the featurization ChemEmbed's CNN consumes.

## When to use

Use when input is a raw MSP file of MS/MS peak lists (e.g. Orbitrap or QTOF scans at several collision energies) that must be turned into fixed-width model inputs for embedding prediction or candidate matching.

## When NOT to use

- Input is already preprocessed model input (e.g. a binned spectrum array or embedding) — re-binning or re-merging would discard information.
- Only a single collision-energy spectrum exists per feature and no merging is intended; the preprocessing step still applies (thresholding/binning) but the merged-spectrum rationale does not.
- Candidate retrieval stage is the target — precursor tolerance (--precursor_tolerance_ppm), not --tolerance, governs filtering there; this skill does not change retrieval.

## Inputs

- MSP file(s) of MS/MS peak lists, with or without SMILES annotations (--input_file_type)
- Adduct polarity flag (+ for [M+H]+, - for [M-H]-)

## Outputs

- Preprocessed, merged spectra as structured DataFrames suitable for CNN input
- Binarized, binned peak arrays (optionally with neutral losses) matching model_positive.bin or model_negative.bin dimensions

## How to apply

Parse the MSP files into structured pandas DataFrames, then merge every MS2 scan of a feature across collision energies into a single spectrum (ChemEmbed's `merge_spectra_by_feature`) to reduce sparsity — the paper found individual collision-energy spectra underperform merged ones and are ~17x slower to train on (95 min vs 5.5 min per epoch). Apply `intensity_threshold` (ChemEmbed used 1) to binarize peak intensities, bin peaks with `--tolerance` (0.01) up to `--max_mz` (700) at the stated resolution (0.01), and optionally append calculated neutral losses. Keep adduct polarity separate: spectra feed either the `[M+H]+` model (model_positive.bin) or the `[M-H]-` model (model_negative.bin), so the `--adduct` flag must match the MSP polarity before prediction.

## Related tools

- **ChemEmbed** (Provides the preprocessing module that converts MSP files to DataFrames, merges spectra by feature, applies intensity thresholding and peak binning prior to CNN prediction) — https://github.com/faizanurv/ChemEmbed
- **pandas** (Structured DataFrame format for parsed MSP spectra (pandas >= 1.0.0))
- **numpy** (Array operations on binned peak intensities (numpy >= 1.18.0))

## Examples

```
python main.py --config config.yaml  # with --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin
```

## Evaluation signals

- All spectra for a feature collapse into exactly one merged row — no feature appears multiple times after merge_spectra_by_feature.
- Every output spectrum matches the expected input dimension implied by --max_mz / resolution binning (ChemEmbed used max_mz 700 at resolution 0.01).
- Peak intensities are binarized at the configured intensity_threshold (ChemEmbed used 1), not continuous.
- The polarity of each preprocessed spectrum matches the --adduct flag so the correct model file (model_positive.bin or model_negative.bin) is invoked downstream.
- Spectra with no peaks surviving the intensity threshold are retained as NA/empty rows, not silently dropped — the README notes unmatched spectra are reported with NA candidates rather than being omitted.

## Limitations

- Merging collision energies and binarizing intensities discards continuous intensity information, which the paper frames as reducing overfitting risk but which limits downstream spectral interpretation.
- Peak binning (--tolerance) has no effect on candidate retrieval — precursor tolerance is a separate parameter — so tuning binning to fix matching recall is a misuse.
- Preprocessing parameters (intensity_threshold, tolerance, resolution, max_mz) must be fixed across train and inference for embedding comparability; changing them breaks the model input contract.
- Only peaks up to max_mz (700 in ChemEmbed) are retained; higher-m/z fragments are discarded.

## Evidence

- [other] ChemEmbed used intensity_threshold=1, tolerance=0.01, resolution=0.01, max_mz=700: [evidence span withheld — anchor status: not_found]
