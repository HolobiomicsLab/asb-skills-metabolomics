---
name: precursor-mass-tolerance-filtering
description: 'Use when ranking candidate molecules for MS/MS spectra: precursor m/z
  must be used to restrict the 520,083-compound reference database before embedding
  similarity comparison.'
when_to_use_negative:
- You are only adjusting peak binning — use --tolerance, which is a peak-binning parameter
  and has no effect on candidate retrieval.
- You need to reproduce pre-v1.1.1 results exactly; --precursor_tolerance_ppm 0 restores
  the legacy 3-decimal bucket, which should not be used for new work.
- The query m/z is low-resolution/QTOF but you keep the 5 ppm default — use 10–20
  ppm instead or recall collapses.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemEmbed
  role: Implements the precursor-mass filter (--precursor_tolerance_ppm) as a candidate-retrieval
    step and precursor m/z post-filter on FAISS results
  repo: github.com/massspecdl/ChemEmbed
- name: RDKit
  role: Derive/validate the reference Precursormz column from SMILES for the requested
    adduct
- name: pandas
  role: Load the reference database (parquet preferred over pickle) and write the
    filtered candidate table
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  - task_002
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:e5459e2c4347c27262cd594f2f1c0ac3090568215346791e89e9aef21380b368
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
  - Mol2vec
  - Python+
  - RDKit
  - numpy >=
  - pandas
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# precursor-mass-tolerance-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Narrows the ChemEmbed candidate pool by retaining only reference compounds whose precursor mass falls within a user-specified ppm window of the measured precursor m/z, before cosine-similarity ranking of predicted vs. reference Mol2vec embeddings.

## When to use

Use when ranking candidate molecules for MS/MS spectra: precursor m/z must be used to restrict the 520,083-compound reference database before embedding similarity comparison.

## When NOT to use

- You are only adjusting peak binning — use --tolerance, which is a peak-binning parameter and has no effect on candidate retrieval.
- You need to reproduce pre-v1.1.1 results exactly; --precursor_tolerance_ppm 0 restores the legacy 3-decimal bucket, which should not be used for new work.
- The query m/z is low-resolution/QTOF but you keep the 5 ppm default — use 10–20 ppm instead or recall collapses.

## Inputs

- MSP spectra file with measured precursor m/z values (with or without SMILES annotations)
- ChemEmbed reference database (chemembed_reference_mol2vec_300d_520k.parquet or .pkl; 520,083 compounds with precursor masses)
- --precursor_tolerance_ppm parameter (default 5 ppm)

## Outputs

- Filtered candidate table (results.csv) with exactly one row per input spectrum, top candidates per spectrum or NA when no reference falls in the window
- Count of unmatched spectra printed to stdout

## How to apply

Parse query precursor m/z values from the input MSP spectra, then keep every reference compound whose precursor mass is within --precursor_tolerance_ppm of the measured precursor; the default is 5 ppm for Orbitrap/high-resolution data and 10–20 ppm for QTOF. Apply the filter after (or as a post-filter to) FAISS/embedding retrieval, preserving ChemEmbed's original matching semantics. Report spectra with no in-window reference as NA candidates rather than omitting them, so the output has exactly one row per input spectrum, and print the count of unmatched spectra. Note that --tolerance is a peak-binning parameter with no effect on candidate retrieval, and that --precursor_tolerance_ppm is 'the single most important parameter for recall'.

## Related tools

- **ChemEmbed** (Implements the precursor-mass filter (--precursor_tolerance_ppm) as a candidate-retrieval step and precursor m/z post-filter on FAISS results) — github.com/massspecdl/ChemEmbed
- **RDKit** (Derive/validate the reference Precursormz column from SMILES for the requested adduct)
- **pandas** (Load the reference database (parquet preferred over pickle) and write the filtered candidate table)

## Examples

```
chemembed --adduct "+" --msp_file_positive spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Over the 520,083-row reference, a ±5 ppm window yields a mean of about 5 candidates per query (99th percentile 12).
- Every input spectrum must appear exactly once in the output; unmatched spectra get NA candidates, not omission.
- Unmatched spectra count is printed and should match the number of NA rows in results.csv.
- Random-sample validation of stored Precursormz values against RDKit-derived masses for the requested adduct.
- Validated SHA-256 checksums for model and reference files (e.g. 41a88ef1... for the parquet) before filtering.

## Limitations

- The legacy --precursor_tolerance_ppm 0 mode rounded the reference mass while truncating the query, failing whenever the fourth decimal was 5 or above — about half of all compounds even with exact query mass; roughly a ±0.0005 Da window, ~1.2 ppm at m/z 400.
- Candidate matching can only return molecules present in the 520,083-compound reference database; compounds outside it cannot be identified regardless of spectral quality.
- Results obtained with a differently sized reference or tolerance are not directly comparable.
