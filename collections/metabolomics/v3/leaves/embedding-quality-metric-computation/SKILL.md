---
name: embedding-quality-metric-computation
description: Use when comparing MS/MS-to-embedding prediction quality across spectral
  variants (individual collision energy, merged, merged+neutral loss) on a test split,
  or when validating that a trained ChemEmbed CNN reproduces reported embedding-quality
  numbers.
when_to_use_negative:
- When the goal is annotation ranking against the 520,083-compound database rather
  than embedding quality on known test compounds — use cosine-similarity candidate
  matching instead.
- When spectra are unmatched or the compound lies outside the 520,083-compound reference
  database, since no reference embedding exists to compare against.
- When comparing embeddings across different adduct modes (positive vs negative),
  because distances reflect polarity mismatch rather than spectral-representation
  quality.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemEmbed
  role: Provides the CNN inference and the preprocessing module used to generate 300-dim
    Mol2vec predictions from MSP spectra before metric computation.
  repo: https://github.com/faizanurv/ChemEmbed
- name: PyTorch
  role: Loads the pre-trained model files (model_positive.bin / model_negative.bin)
    and runs batched inference to obtain embeddings.
- name: pandas >= 1.0.0
  role: Loads the reference database (parquet or pkl) and holds per-spectrum embeddings
    for vectorised metric computation.
- name: numpy >= 1.18.0
  role: Performs the vector arithmetic for Euclidean distance and cosine similarity
    between predicted and reference embeddings.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:62804741d00157251a848b3877ae8cc7e4d01f1de948e1f746d6832ddac687da
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
  - numpy >=
  - numpy >= 1.18.0
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# embedding-quality-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Euclidean distance and cosine similarity between CNN-predicted 300-dimensional Mol2vec embeddings and reference embeddings from the 520,083-compound ChemEmbed database, to quantify how well spectra (individual, merged, or merged+neutral-loss) map to molecular structure.

## When to use

Use when comparing MS/MS-to-embedding prediction quality across spectral variants (individual collision energy, merged, merged+neutral loss) on a test split, or when validating that a trained ChemEmbed CNN reproduces reported embedding-quality numbers.

## When NOT to use

- When the goal is annotation ranking against the 520,083-compound database rather than embedding quality on known test compounds — use cosine-similarity candidate matching instead.
- When spectra are unmatched or the compound lies outside the 520,083-compound reference database, since no reference embedding exists to compare against.
- When comparing embeddings across different adduct modes (positive vs negative), because distances reflect polarity mismatch rather than spectral-representation quality.

## Inputs

- Predicted 300-dimensional Mol2vec embeddings from a ChemEmbed CNN model (model_positive.bin or model_negative.bin)
- Reference Mol2vec 300-dim embeddings with SMILES, InChIKeys and precursor masses (chemembed_reference_mol2vec_300d_520k.parquet or .pkl, 520,083 compounds)
- Preprocessed test-split MSP spectra (structured DataFrames with intensity_threshold=1, tolerance=0.01, resolution=0.01, max_mz=700)

## Outputs

- Per-spectrum Euclidean distance between predicted and reference Mol2vec embeddings
- Per-spectrum cosine similarity between predicted and reference Mol2vec embeddings
- Aggregated mean Euclidean distance and mean cosine similarity per spectral variant (individual, merged, merged+neutral loss)

## How to apply

Run batched CNN inference (batch_size=32) on the preprocessed test-split MSP spectra (intensity_threshold=1, tolerance=0.01, resolution=0.01, max_mz=700) to obtain 300-dim Mol2vec predictions for each spectral variant. For each prediction, compute the Euclidean distance and cosine similarity against the reference Mol2vec embedding of the same compound from the 520,083-compound parquet/pkl database. Restrict comparisons to the same adduct mode (positive uses model_positive.bin, negative uses model_negative.bin) so that distance values reflect spectral representation quality rather than polarity mismatch. Aggregate per-variant mean Euclidean distance and mean cosine similarity; reported test-split values in positive mode are distance 40 (individual) vs 24.5 (merged) vs 23.1 (merged+neutral loss) and cosine 0.93 vs 0.96 vs 0.97. Lower distance and higher cosine indicate that the enriched spectrum better captures structural properties of the molecule.

## Related tools

- **ChemEmbed** (Provides the CNN inference and the preprocessing module used to generate 300-dim Mol2vec predictions from MSP spectra before metric computation.) — https://github.com/faizanurv/ChemEmbed
- **PyTorch** (Loads the pre-trained model files (model_positive.bin / model_negative.bin) and runs batched inference to obtain embeddings.)
- **pandas >= 1.0.0** (Loads the reference database (parquet or pkl) and holds per-spectrum embeddings for vectorised metric computation.)
- **numpy >= 1.18.0** (Performs the vector arithmetic for Euclidean distance and cosine similarity between predicted and reference embeddings.)

## Examples

```
python main.py --config config.yaml  # config.yaml points to model_positive.bin and chemembed_reference_mol2vec_300d_520k.parquet for batch inference, then compute cdist(pred, ref, metric='euclidean') and cosine similarities
```

## Evaluation signals

- Mean Euclidean distance should decrease monotonically across variants in positive mode: 40 (individual) → 24.5 (merged) → 23.1 (merged+neutral loss).
- Mean cosine similarity should increase monotonically: 0.93 → 0.96 → 0.97 for the same sequence.
- Cosine similarity must fall in the expected range for a 300-dim embedding (≈0.9+ on the test split); values near 0 or negative indicate a polarity or preprocessing mismatch.
- Per-spectrum distances must be finite (no NaN/inf) — unmatched spectra should be excluded or flagged before aggregation, not silently averaged.
- The 520,083-compound reference database must load with the expected row count; verify sha256 against the published values before computing metrics.

## Limitations

- Metrics are computed against a fixed 520,083-compound reference; results are not directly comparable when using a differently sized or differently versioned database.
- The reported thresholds (top-1 > 42%, top-5 > 76%) are specific to the paper's test split and cannot be transferred to other datasets without recalibration.
- The CNN predicts embeddings only for the adduct mode of its weights file; mixing modes invalidates the comparison.
- Candidate matching can only return molecules already present in the reference database, so compounds outside it cannot be identified regardless of spectral quality.

## Evidence
