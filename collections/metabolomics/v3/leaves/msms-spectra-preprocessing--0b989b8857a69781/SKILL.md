---
name: msms-spectra-preprocessing
description: Use when input is raw MSP files of tandem MS spectra (possibly split
  across multiple collision energies) and you need model-ready spectra arrays for
  CNN-based metabolite identification with ChemEmbed.
when_to_use_negative:
- The spectra are already merged, binarized model inputs — re-binning or re-binarizing
  could destroy peak information
- You need to adjust candidate retrieval parameters — precursor filtering is controlled
  by --precursor_tolerance_ppm, not by the spectral preprocessing parameters
- You are reproducing legacy pre-1.1.1 results where the exact 3-decimal precursor
  bucket was used — in that case use --precursor_tolerance_ppm 0 rather than altering
  preprocessing
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ChemEmbed
  role: Provides the data preprocessing module that converts MSP files to structured
    DataFrames, merges collision-energy scans, binarizes intensities, and applies
    peak binning for model input
  repo: https://github.com/faizanurv/ChemEmbed.git
- name: pandas
  role: Used by the preprocessing step to build and manipulate structured DataFrames
    from MSP files
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:07a89445874b0efc0a70ca16f06bd67e55017769cae28ba8dcadfe78e3c3c027
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  repo_url: https://github.com/faizanurv/ChemEmbed.git
  tools:
  - ChemEmbed
  - Mol2vec
  - Python+
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# msms-spectra-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess MS/MS spectra for ChemEmbed's CNN by merging multi-collision-energy MS2 scans, binarizing intensities, applying peak binning, and converting MSP files into model-ready arrays. This enhanced-spectrum format is what the pretrained CNN expects before predicting 300-dimensional Mol2vec embeddings.

## When to use

Use when input is raw MSP files of tandem MS spectra (possibly split across multiple collision energies) and you need model-ready spectra arrays for CNN-based metabolite identification with ChemEmbed.

## When NOT to use

- The spectra are already merged, binarized model inputs — re-binning or re-binarizing could destroy peak information
- You need to adjust candidate retrieval parameters — precursor filtering is controlled by --precursor_tolerance_ppm, not by the spectral preprocessing parameters
- You are reproducing legacy pre-1.1.1 results where the exact 3-decimal precursor bucket was used — in that case use --precursor_tolerance_ppm 0 rather than altering preprocessing

## Inputs

- MSP files of MS/MS spectra, with or without SMILES annotations (input_file_type 'with_smiles' or 'without_smiles')
- YAML configuration file (config.yaml) specifying intensity threshold, resolution, tolerance, max_mz, and input/output paths
- Polarity information ([M+H]+ or [M-H]-) to select the correct preprocessing/model path

## Outputs

- Structured DataFrames of enhanced (merged, binarized, peak-binned) spectra ready for CNN model input
- Preprocessed spectra arrays feeding 300-dimensional Mol2vec embedding prediction

## How to apply

Convert the MSP files to structured DataFrames with ChemEmbed's data preprocessing, selecting the correct input_file_type ('without_smiles' for external benchmarks like CASMI 2022, 'with_smiles' when annotations are present). Merge the MS2 scans of each feature into a single spectrum (merge_spectra_by_feature), since the article shows merged spectra both reduce sparsity and train ~17x faster than individual collision-energy spectra. Binarize intensities using the intensity_threshold parameter and apply peak binning with the --tolerance and --resolution settings (e.g., tolerance 0.01, resolution 0.01, max_mz 700 for CASMI 2022). Output the preprocessed arrays so the polarity-appropriate pretrained model (model_positive.bin for [M+H]+, model_negative.bin for [M-H]-) can be applied directly. Note that --tolerance is a peak-binning parameter only and has no effect on candidate retrieval, which is governed separately by --precursor_tolerance_ppm.

## Related tools

- **ChemEmbed** (Provides the data preprocessing module that converts MSP files to structured DataFrames, merges collision-energy scans, binarizes intensities, and applies peak binning for model input) — https://github.com/faizanurv/ChemEmbed.git
- **pandas** (Used by the preprocessing step to build and manipulate structured DataFrames from MSP files)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Each input spectrum (or feature) yields exactly one merged spectrum after merge_spectra_by_feature; one row per input spectrum is expected downstream
- Intensities are binarized according to the configured intensity_threshold (e.g., threshold 1 in the CASMI 2022 workflow) — no continuous intensities remain
- Peak bins respect the configured resolution/tolerance (e.g., 0.01/0.01) and max_mz cutoff (e.g., 700 m/z)
- The preprocessed arrays are compatible with the polarity-specific pretrained weights (model_positive.bin / model_negative.bin) without shape errors at CNN inference
- Settings match the article's CASMI 2022 parameters (input_file_type 'without_smiles', intensity_threshold 1, tolerance 0.01, resolution 0.01, max_mz 700) when reproducing that benchmark

## Limitations

- Training or inference on individual collision-energy spectra underperforms merged spectra because individual spectra are sparser, so preprocessing must merge scans per feature
- The reference database covers 520,083 compounds; compounds outside it cannot be identified regardless of spectral quality, so preprocessing from a different reference scope is not directly comparable
- --tolerance is only a peak-binning parameter and has no effect on candidate retrieval, a common confusion point

## Evidence
