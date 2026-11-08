---
name: collision-energy-spectra-consolidation
description: Use when preparing enhanced MS/MS training or query inputs from MSP-derived
  spectra where each compound has multiple MS2 scans at different collision energies,
  and a single binarized merged spectrum with neutral losses is needed for model input.
when_to_use_negative:
- 'When the goal is candidate retrieval accuracy tuning: `--tolerance` is a peak-binning
  parameter and has no effect on candidate retrieval, which is governed by `--precursor_tolerance_ppm`
  instead.'
- When input spectra already exist as a single merged spectrum per compound with binarized
  intensities — re-applying the merge would be redundant.
- When preserving native relative intensity information is required for downstream
  tools that expect continuous intensity values, since binarization discards magnitude.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ChemEmbed
  role: Provides the data preprocessing module that merges multiple-collision-energy
    spectra, appends neutral losses, and binarizes intensities for CNN input
  repo: https://github.com/faizanurv/ChemEmbed
- name: pandas >= 1.0.0
  role: Holds the structured DataFrame representation of MSP-derived spectra during
    merging and binarization
- name: numpy >= 1.18.0
  role: Numerical computation for fragment/neutral-loss m/z arrays and intensity binarization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:dc1bde37f3e10a3d7be250fed56688a5ade3afafe1c9dac601098d5635f225f0
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
  - numpy >=
  - numpy >= 1.18.0
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# collision-energy-spectra-consolidation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consolidates per-compound MS/MS spectra acquired across multiple collision energies into a single representative spectrum, adding calculated neutral losses and binarizing intensities, as implemented in ChemEmbed to reduce sparsity and overfitting in CNN-based metabolite identification.

## When to use

Use when preparing enhanced MS/MS training or query inputs from MSP-derived spectra where each compound has multiple MS2 scans at different collision energies, and a single binarized merged spectrum with neutral losses is needed for model input.

## When NOT to use

- When the goal is candidate retrieval accuracy tuning: `--tolerance` is a peak-binning parameter and has no effect on candidate retrieval, which is governed by `--precursor_tolerance_ppm` instead.
- When input spectra already exist as a single merged spectrum per compound with binarized intensities — re-applying the merge would be redundant.
- When preserving native relative intensity information is required for downstream tools that expect continuous intensity values, since binarization discards magnitude.

## Inputs

- MSP files with MS/MS spectra per compound acquired at multiple collision energies (with or without SMILES annotations)
- YAML configuration file specifying intensity_threshold, tolerance (peak binning), and input/output paths

## Outputs

- One merged, binarized spectrum table row per compound, containing combined fragment ions from all collision energies plus appended neutral-loss peaks (0/1 intensities)
- Structured pandas DataFrame representation of the preprocessed spectra ready for CNN model input

## How to apply

Load per-compound MS/MS spectra spanning multiple collision energies from MSP files into structured pandas DataFrames using ChemEmbed's preprocessing module (numpy/pandas >= 1.18.0/1.0.0). Apply the `merge_spectra_by_feature`-style collapse so every MS2 scan of a compound/feature is combined into one representative spectrum. Compute and append calculated neutral-loss peaks (precursor minus fragment m/z) to the merged spectrum. Binarize all peak intensities to 0/1, respecting the configured `--tolerance` peak-binning parameter and intensity threshold from the YAML config; `--tolerance` affects peak alignment only, not candidate retrieval. Write one merged, binarized spectrum per compound to a spectrum table, validating that each compound yields exactly one row containing both fragment and neutral-loss peaks. The rationale is that individual collision-energy spectra are too sparse for learning — merged spectra cut one training epoch from over 95 minutes to 5.5 minutes and reduce overfitting risk.

## Related tools

- **ChemEmbed** (Provides the data preprocessing module that merges multiple-collision-energy spectra, appends neutral losses, and binarizes intensities for CNN input) — https://github.com/faizanurv/ChemEmbed
- **pandas >= 1.0.0** (Holds the structured DataFrame representation of MSP-derived spectra during merging and binarization)
- **numpy >= 1.18.0** (Numerical computation for fragment/neutral-loss m/z arrays and intensity binarization)

## Examples

```
python main.py --config config.yaml   # with config.yaml setting intensity_threshold, tolerance, and msp input paths for merged, binarized spectra
```

## Evaluation signals

- Each compound yields exactly one merged spectrum row (no compound appears with multiple rows).
- All peak intensity entries in the output spectrum table are binarized to 0 or 1 — no continuous values remain.
- The merged spectrum contains both fragment-ion peaks and appended neutral-loss peaks (precursor minus fragment m/z).
- Peak alignment respects the configured `--tolerance` binning window so that peaks across collision energies are not duplicated at near-identical m/z.
- Downstream training shows the expected performance gain: one epoch should complete in roughly 5.5 minutes rather than over 95 minutes compared to per-collision-energy inputs.

## Limitations

- Individual collision-energy spectra have higher sparsity that complicates learning; skipping the merge step leads to underperforming models and ~17x slower epochs.
- MSP files without SMILES annotations are handled via a separate `--input_file_type without_smiles` configuration path.
- The legacy `--precursor_tolerance_ppm 0` three-decimal bucket comparison rounded the reference but truncated the query, failing for roughly half of compounds even with exact query masses — a pitfall in the surrounding pipeline, not the merge itself, but relevant when feeding merged spectra into candidate matching.

## Evidence

- [discussion] merge multiple CE and binarize: "Our tool, ChemEmbed, merges data from multiple collision energies and binarizes the intensities, simplifying the input and reducing overfitting risks"
- [methods] merge_spectra_by_feature collapse: "By default `merge_spectra_by_feature` collapses every MS2 scan of a feature into a single spectrum."
- [discussion] individual spectra underperform and are slow: "training neural networks with individual collision energy spectra per molecule underperforms compared to using a single merged spectrum. The higher sparsity of individual spectra complicates learning"
- [discussion] one epoch speedup: "training one epoch with individual spectra took over 95 minutes, compared to just 5.5 minutes with merged spectra"
