---
name: molecular-embedding-prediction
description: Use when you have MSP-format MS/MS spectra (positive or negative ionization)
  and need to convert them into molecular embeddings to rank candidate structures,
  e.g. for metabolite identification or annotation of unannotated spectra.
when_to_use_negative:
- Your target compounds may lie outside the 520,083-compound reference database —
  ChemEmbed cannot identify structures absent from it regardless of spectral quality
- You need to compare results against a differently sized reference database — results
  obtained with a smaller or larger reference are not directly comparable
- You have raw mzML/centroided profile data rather than MSP peak lists — ChemEmbed's
  preprocessing expects MSP files converted to structured DataFrames
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemEmbed
  role: 'End-to-end pipeline: MSP preprocessing, merged-spectrum enhancement with
    neutral losses, CNN embedding prediction, and cosine-similarity candidate matching
    against the Mol2vec reference database'
  repo: https://github.com/faizanurv/ChemEmbed
- name: PyTorch
  role: Runs the pre-trained CNN inference (batch_size=32) that maps enhanced MS/MS
    spectra to 300-dim Mol2vec embeddings
- name: Mol2vec
  role: Supplies the 300-dimensional unsupervised molecular embeddings that define
    both the CNN prediction target and the reference-database representation
- name: RDKit
  role: Converts InChI identifiers to SMILES format when building reference entries
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:21a7d019fba378f0e5efbf8ccb6737a19e4663161b5c67d38f510aecbb4a9c1d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# molecular-embedding-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs ChemEmbed's pre-trained CNN to predict 300-dimensional Mol2vec embeddings from enhanced (merged, neutral-loss-enriched) MS/MS spectra and ranks candidate metabolites by cosine similarity against a 520,083-compound reference database.

## When to use

Use when you have MSP-format MS/MS spectra (positive or negative ionization) and need to convert them into molecular embeddings to rank candidate structures, e.g. for metabolite identification or annotation of unannotated spectra.

## When NOT to use

- Your target compounds may lie outside the 520,083-compound reference database — ChemEmbed cannot identify structures absent from it regardless of spectral quality
- You need to compare results against a differently sized reference database — results obtained with a smaller or larger reference are not directly comparable
- You have raw mzML/centroided profile data rather than MSP peak lists — ChemEmbed's preprocessing expects MSP files converted to structured DataFrames

## Inputs

- MSP file of MS/MS spectra with SMILES annotations (input_file_type 'with_smiles') or without SMILES ('without_smiles', e.g. CASMI 2022 files)
- Adduct/ionization mode ('+' for [M+H]+ or '-' for [M-H]-)
- Pre-trained CNN weights: model_positive.bin / model_negative.bin (~1130 MB each, from the v1.0.1 GitHub release)
- Reference database: chemembed_reference_mol2vec_300d_520k.pkl or .parquet (520,083 compounds, 300-dim Mol2vec embeddings, precursor masses, SMILES, InChIKeys)
- config.yaml with paths and parameters (intensity_threshold, tolerance, resolution, max_mz, top_n_candidates, precursor_tolerance_ppm)

## Outputs

- CSV of top-N candidate molecules per input spectrum with cosine-similarity rankings (results.csv), one row per input spectrum including NA candidates for unmatched masses
- 300-dimensional Mol2vec embedding per predicted spectrum
- Count/log of unmatched spectra (no reference within precursor tolerance)

## How to apply

Preprocess MSP files into structured DataFrames with ChemEmbed's preprocessing module (input_file_type 'with' or 'without_smiles', intensity_threshold=1, tolerance=0.01, resolution=0.01, max_mz=700). Load the polarity-appropriate CNN weights (model_positive.bin for [M+H]+, model_negative.bin for [M-H]-) and run batched inference (batch_size=32) to predict 300-dim Mol2vec embeddings from the merged spectra with calculated neutral losses. Match predictions against the 520,083-compound reference database (chemembed_reference_mol2vec_300d_520k.pkl or .parquet) keeping every reference compound whose precursor mass is within --precursor_tolerance_ppm (default 5 for Orbitrap; 10–20 for QTOF), then rank candidates by cosine similarity. The merged-spectrum enrichment is the rationale: training/inference on merged spectra improves embedding quality over individual collision-energy spectra (mean cosine similarity 0.93→0.97 in positive mode) and is ~17× faster per epoch (5.5 vs 95 min).

## Related tools

- **ChemEmbed** (End-to-end pipeline: MSP preprocessing, merged-spectrum enhancement with neutral losses, CNN embedding prediction, and cosine-similarity candidate matching against the Mol2vec reference database) — https://github.com/faizanurv/ChemEmbed
- **PyTorch** (Runs the pre-trained CNN inference (batch_size=32) that maps enhanced MS/MS spectra to 300-dim Mol2vec embeddings)
- **Mol2vec** (Supplies the 300-dimensional unsupervised molecular embeddings that define both the CNN prediction target and the reference-database representation)
- **RDKit** (Converts InChI identifiers to SMILES format when building reference entries)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- On the test split, mean cosine similarity between predicted and reference Mol2vec embeddings should be ≥0.97 in positive mode with merged spectra + neutral losses (vs 0.93 for individual spectra); mean Euclidean distance should fall to ~23 (vs 40)
- Top-5 correct-compound ranking should reach or approach 33% (positive) and 30% (negative) on the filtered CASMI 2022 benchmark — clearly above SIRIUS 6.0.0's 28% and 17%
- On the internal test dataset, top-ranked annotations should exceed 42% and top-5 exceed 76% of cases
- Results file must contain exactly one row per input spectrum, including NA-candidate rows for spectra with no reference mass within the precursor tolerance window
- Reference DB, model files, and hashes must match the v1.0.1 release sha256 values (e.g. b376f329… for model_positive.bin) before running

## Limitations

- Candidate matching is limited to the 520,083 compounds in the reference database; compounds outside it cannot be identified
- The legacy --precursor_tolerance_ppm 0 (exact 3-decimal bucket) failed about half of compounds even with exact query masses and is retained only for reproducing pre-1.1.1 results
- --tolerance is a peak-binning parameter and does not affect candidate retrieval
- pip install chemembed pulls ~5 GB of PyTorch/CUDA libraries; a small /tmp may need TMPDIR redirection
- Results obtained with a differently sized reference database are not directly comparable

## Evidence
