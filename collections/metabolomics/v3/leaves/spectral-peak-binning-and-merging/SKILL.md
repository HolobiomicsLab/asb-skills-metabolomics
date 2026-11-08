---
name: spectral-peak-binning-and-merging
description: Use when you have per-compound MS/MS spectra (e.g. from MSP files) acquired
  at several collision energies and need to produce the merged, binarized, neutral-loss-enriched
  spectrum that ChemEmbed's CNN expects as model input.
when_to_use_negative:
- When you already possess a single, well-populated spectrum per compound (merging
  is unnecessary and would discard collision-energy-resolved information)
- When you need absolute intensity information downstream — binarization discards
  it by design
- When your goal is candidate retrieval tuning — --tolerance is a peak-binning parameter
  and has no effect on candidate retrieval
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ChemEmbed
  role: Hosts the preprocessing module that converts MSP files to structured DataFrames
    and performs the spectral merging, neutral-loss enrichment, and binarization
  repo: github.com/massspecdl/ChemEmbed
- name: pandas >= 1.0.0
  role: Used to hold the structured DataFrames produced from MSP files and carry the
    merged, binarized spectra
- name: numpy >= 1.18.0
  role: Numeric operations for peak binning, neutral-loss calculation, and intensity
    binarization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:88b9f8a04b537b2fa19609ef8bf8e39bad0811a364d17568a9b6ece00a1a437e
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spectral-peak-binning-and-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs ChemEmbed's enhanced MS/MS input by collapsing every MS2 scan of a compound across multiple collision energies into a single merged spectrum, appending calculated neutral-loss peaks, binarizing intensities, and peak-binning m/z values — the preprocessing that makes spectra learnable and dramatically reduces training sparsity.

## When to use

Use when you have per-compound MS/MS spectra (e.g. from MSP files) acquired at several collision energies and need to produce the merged, binarized, neutral-loss-enriched spectrum that ChemEmbed's CNN expects as model input.

## When NOT to use

- When you already possess a single, well-populated spectrum per compound (merging is unnecessary and would discard collision-energy-resolved information)
- When you need absolute intensity information downstream — binarization discards it by design
- When your goal is candidate retrieval tuning — --tolerance is a peak-binning parameter and has no effect on candidate retrieval

## Inputs

- MSP file(s) containing per-compound MS/MS spectra acquired at multiple collision energies
- Precursor m/z for each compound
- Intensity threshold parameter (e.g. config.yaml value)
- Peak binning tolerance (--tolerance) and m/z resolution settings

## Outputs

- One merged, binarized spectrum per compound, containing both fragment-ion peaks and calculated neutral-loss peaks
- Structured DataFrames suitable for CNN model input

## How to apply

1) Load the per-compound MS/MS spectra spanning multiple collision energies from MSP-derived data into pandas/numpy DataFrames. 2) Apply the merge_spectra_by_feature collapse so every MS2 scan of a compound/feature combines into one representative spectrum. 3) Compute calculated neutral-loss peaks (precursor minus fragment m/z) and append them to the merged spectrum. 4) Binarize all peak intensities to presence/absence, respecting the configured intensity threshold. 5) Peak-bin the merged spectrum using the --tolerance parameter, which affects only peak alignment, not candidate retrieval. The rationale is that individual collision-energy spectra are sparse and hard to learn from: ChemEmbed's authors found that training with merged spectra cut one epoch from over 95 minutes to 5.5 minutes (~17x faster) and reduced overfitting risk.

## Related tools

- **ChemEmbed** (Hosts the preprocessing module that converts MSP files to structured DataFrames and performs the spectral merging, neutral-loss enrichment, and binarization) — github.com/massspecdl/ChemEmbed
- **pandas >= 1.0.0** (Used to hold the structured DataFrames produced from MSP files and carry the merged, binarized spectra)
- **numpy >= 1.18.0** (Numeric operations for peak binning, neutral-loss calculation, and intensity binarization)

## Examples

```
python main.py --config config.yaml  # config.yaml sets intensity_threshold, resolution, and --tolerance for peak binning and binarization
```

## Evaluation signals

- Check that each compound yields exactly one merged spectrum row after merge_spectra_by_feature collapse
- Verify all intensity values in the output spectrum are binarized (0/1), not raw counts
- Confirm neutral-loss peaks are present: the merged spectrum should contain peaks corresponding to precursor-minus-fragment m/z values in addition to original fragments
- Confirm that the peak-binning tolerance applied matches the configured --tolerance value and that no fragment peaks were merged across the tolerance window
- Verify downstream training-time reduction: one training epoch on merged spectra should take roughly 5.5 minutes versus over 95 minutes for individual spectra on comparable hardware

## Limitations

- The reference database's peak-binning tolerance parameter (--tolerance) does not affect candidate retrieval — it only shapes the input spectra; confusing these roles can mislead parameter tuning
- Binarization discards absolute intensity information, which may be undesirable if downstream analysis requires quantitative spectral data
- Merging collapses all collision energies into one spectrum, so energy-resolved fragmentation patterns are lost; training with individual collision energy spectra is possible but underperforms and is ~17x slower

## Evidence
