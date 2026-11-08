---
name: msms-embedding-prediction
description: Use when annotating MSP-format MS/MS spectra (MSP files with or without
  SMILES) for a chosen polarity whose features have precursor masses and known adducts,
  and you need top-N candidate compound IDs from embeddings rather than spectral-library
  dot-product matching.
when_to_use_negative:
- The target compound is not present in the 520,083-compound reference database —
  candidate matching can only return molecules in the database, so out-of-reference
  compounds cannot be identified regardless of spectral quality
- You need atom-level or formula-level annotation rather than structural embedding
  candidates — ChemEmbed returns top-N molecule candidates by embedding similarity,
  not de novo formulas
- Your data is raw vendor binary (e.g. Thermo RAW, mzML) rather than MSP-converted
  spectra — ChemEmbed's preprocessing expects MSP files converted to DataFrames first
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ChemEmbed
  role: 'Core pipeline: preprocesses MSP spectra, runs CNN inference to predict Mol2vec
    embeddings, and matches them by cosine similarity against the reference DB'
  repo: https://github.com/faizanurv/ChemEmbed
- name: Mol2vec
  role: Provides the 300-dimensional continuous molecular-structure embeddings that
    the CNN is trained to predict from enhanced MS/MS spectra
- name: FAISS
  role: Vector index used for fast cosine-similarity search over the 520,083-compound
    reference embeddings, searched in a single GPU call per sample with precursor
    m/z post-filter
- name: RDKit
  role: Not required in this embedding-prediction path; ChemEmbed used RDKit upstream
    to convert InChI identifiers to SMILES when building the reference database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:74aecea693168b3252f33bfa5997a2029bda2effe7944bd9ddb2905b6542e6a0
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
  - FAISS
  - Mol2vec
  - Python+
  - RDKit
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# MS/MS embedding prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs ChemEmbed's pre-trained positive- and negative-mode CNNs over merged, binarized MS/MS spectra to predict 300-dimensional Mol2vec embeddings, which are matched by cosine similarity against a 520,083-compound reference database to retrieve top candidate metabolite identifications.

## When to use

Use when annotating MSP-format MS/MS spectra (MSP files with or without SMILES) for a chosen polarity whose features have precursor masses and known adducts, and you need top-N candidate compound IDs from embeddings rather than spectral-library dot-product matching.

## When NOT to use

- The target compound is not present in the 520,083-compound reference database — candidate matching can only return molecules in the database, so out-of-reference compounds cannot be identified regardless of spectral quality
- You need atom-level or formula-level annotation rather than structural embedding candidates — ChemEmbed returns top-N molecule candidates by embedding similarity, not de novo formulas
- Your data is raw vendor binary (e.g. Thermo RAW, mzML) rather than MSP-converted spectra — ChemEmbed's preprocessing expects MSP files converted to DataFrames first

## Inputs

- MSP-format MS/MS spectra file (positive or negative mode, with or without SMILES annotations)
- Pre-trained CNN model weights (model_positive.bin or model_negative.bin)
- Reference database of Mol2vec 300-dim embeddings with precursor masses (chemembed_reference_mol2vec_300d_520k.pkl or .parquet, 520,083 compounds)
- config.yaml with intensity threshold, resolution, tolerance, and top_n_candidates
- Precursor m/z of each feature and adduct polarity (+/-)

## Outputs

- Predicted 300-dimensional Mol2vec embeddings per spectrum
- Ranked top-N candidate compounds per spectrum with cosine similarity scores
- Results CSV (prediction_results) with one row per input spectrum, NA candidates for unmatched spectra
- Count of spectra with no reference candidate inside the precursor window

## How to apply

Convert the MSP file to structured DataFrames with ChemEmbed's data preprocessing module, merge all MS2 scans of each feature into one spectrum and binarize intensities, then load the pre-trained CNN for the polarity (model_positive.bin for [M+H]+, model_negative.bin for [M-H]-) and run batched GPU inference to predict 300-dim Mol2vec embeddings. Retrieve candidates by keeping reference compounds whose precursor mass is within --precursor_tolerance_ppm of the measured precursor (5 ppm for Orbitrap-class, 10-20 ppm for QTOF, never 0 for new work), then match embeddings by cosine similarity via FAISS and return the top_n_candidates (e.g. 5) per spectrum. Note --tolerance is only a peak-binning parameter and does not affect candidate retrieval; spectra with no reference in the mass window are emitted with NA candidates so one row per input spectrum is guaranteed. Optionally post-filter FAISS results by precursor m/z to preserve ChemEmbed's matching semantics.

## Related tools

- **ChemEmbed** (Core pipeline: preprocesses MSP spectra, runs CNN inference to predict Mol2vec embeddings, and matches them by cosine similarity against the reference DB) — https://github.com/faizanurv/ChemEmbed
- **Mol2vec** (Provides the 300-dimensional continuous molecular-structure embeddings that the CNN is trained to predict from enhanced MS/MS spectra)
- **FAISS** (Vector index used for fast cosine-similarity search over the 520,083-compound reference embeddings, searched in a single GPU call per sample with precursor m/z post-filter)
- **RDKit** (Not required in this embedding-prediction path; ChemEmbed used RDKit upstream to convert InChI identifiers to SMILES when building the reference database)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- One output row per input spectrum in the results CSV, including NA candidates for unmatched masses
- Precursor tolerance set to 5 ppm (high-res) / 10-20 ppm (QTOF), not 0 — the legacy 3-decimal bucket silently fails ~half of exact-mass compounds
- SHA256 of model_positive.bin, model_negative.bin, and the reference DB matches the published values (e.g. b376f329f1b3759e598c9e1fd2e6b252d9f63ed2316baaff3b57be387b0b8c73 for model_positive.bin)
- Embedding dimensionality is exactly 300 and matches the reference DB's 300-dim Mol2vec vectors
- On the paper's ARUS validation, top-5 candidates from 40 expert-reviewed spectra (10 per matrix×mode) reproduce 24 confirmed identifications (18 positive, 6 negative)

## Limitations

- Candidate retrieval can only return compounds in the 520,083-compound reference DB; out-of-DB compounds are structurally unreachable regardless of spectral quality
- Results from a differently sized reference are not directly comparable to the published benchmarks (76% top-5, 42% top-1)
- Setting --precursor_tolerance_ppm 0 restores a legacy 3-decimal bucket filter that fails when the fourth decimal is ≥5 — about half of compounds even with exact query masses
- --tolerance is a peak-binning parameter only; changing it does not loosen candidate retrieval
- MSP input must have a defined adduct/polarity; the model is polarity-specific (separate [M+H]+ and [M-H]- weights)

## Evidence
