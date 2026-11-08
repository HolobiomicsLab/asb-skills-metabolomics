---
name: metabolite-identification-annotation
description: Use when MS/MS spectra (MSP/MGF) remain unmatched to spectral libraries
  (e.g. ARUS spectra) and candidate compound annotations are needed, with formulas
  assigned (e.g. BUDDY) and a precursor m/z available for filtering.
when_to_use_negative:
- The target compound is not in the 520,083-compound reference database — candidate
  matching can only return molecules in the database, regardless of spectral quality
- Only individual collision-energy spectra are available without the ability to merge
  scans per feature — training/prediction with individual CE spectra underperforms
  merged spectra and inflates sparsity
- You need de novo formula/structure inference rather than library-style candidate
  ranking against a fixed reference (e.g. use SIRIUS instead)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ChemEmbed
  role: 'End-to-end pipeline: MSP preprocessing, merged-spectrum binarization, CNN
    embedding prediction, and cosine-similarity candidate matching'
  repo: https://github.com/faizanurv/ChemEmbed
- name: pandas >= 1.0.0
  role: Structured DataFrames for MSP files and result compilation
- name: RDKit
  role: Converted InChI identifiers to SMILES format when building the reference database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:d28057fdbb6c771ebf6714b2234a34fe5d11b67027552ef850a91c75ca7527e7
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
  - Python+
  - RDKit
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# metabolite-identification-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate previously unidentified MS/MS spectra by predicting 300-dimensional Mol2vec embeddings from merged, binarized spectra with pre-trained CNN models, then matching embeddings against a 520,083-compound reference database by cosine similarity with precursor-mass pre-filtering. This is the ChemEmbed pipeline demonstrated on the ARUS plasma/urine validation set, where expert review of top-5 candidates confirmed 24 compounds.

## When to use

Use when MS/MS spectra (MSP/MGF) remain unmatched to spectral libraries (e.g. ARUS spectra) and candidate compound annotations are needed, with formulas assigned (e.g. BUDDY) and a precursor m/z available for filtering.

## When NOT to use

- The target compound is not in the 520,083-compound reference database — candidate matching can only return molecules in the database, regardless of spectral quality
- Only individual collision-energy spectra are available without the ability to merge scans per feature — training/prediction with individual CE spectra underperforms merged spectra and inflates sparsity
- You need de novo formula/structure inference rather than library-style candidate ranking against a fixed reference (e.g. use SIRIUS instead)

## Inputs

- MSP/MGF MS/MS spectra files for positive ([M+H]+) and negative ([M-H]-) modes (e.g. ARUS plasma/urine spectra with BUDDY-assigned formulas)
- Pre-trained CNN model weights: model_positive.bin and model_negative.bin (1129.6 MB each, v1.0.1 release)
- Reference database of 520,083 compounds with 300-dim Mol2vec embeddings: chemembed_reference_mol2vec_300d_520k.parquet (or .pkl)
- config.yaml specifying input/output paths, intensity_threshold, tolerance (peak binning), precursor_tolerance_ppm, and top_n_candidates

## Outputs

- prediction_results CSV with one row per input spectrum, listing top_n_candidates (default 5) per spectrum ranked by cosine similarity, separated by ionization mode
- NA-candidate rows for spectra with no reference compound within the precursor mass window
- Count of unmatched spectra printed during the run

## How to apply

Load the MSP/MGF spectra and convert them to structured DataFrames with ChemEmbed's data preprocessing module, merging all MS/MS scans of each feature into a single spectrum across collision energies and binarizing intensities (spectra with intensities binarized; --tolerance only affects peak binning, not retrieval). Select the pre-trained CNN matching the ionization mode — model_positive.bin for [M+H]+ or model_negative.bin for [M-H]- — and run batched inference (GPU batch_size 32) to predict 300-dimensional Mol2vec embeddings. Filter the 520,083-compound reference database to compounds whose precursor mass is within --precursor_tolerance_ppm of the measured precursor (5 ppm default for high-resolution Orbitrap data, 10–20 for QTOF), then search predicted embeddings against the reference (FAISS index) by cosine similarity, retrieving top_n_candidates = 5 candidates per spectrum. Spectra with no reference inside the mass window are reported with NA candidates rather than omitted, so the results file has one row per input spectrum. Finally, cross-check the top-5 candidate lists against assigned formulas (e.g. BUDDY) and expert manual review to confirm compound identities, as in the ARUS validation where 40 randomly selected spectra (10 per matrix and ionization mode) yielded 24 confirmed compounds (18 positive, 6 negative).

## Related tools

- **ChemEmbed** (End-to-end pipeline: MSP preprocessing, merged-spectrum binarization, CNN embedding prediction, and cosine-similarity candidate matching) — https://github.com/faizanurv/ChemEmbed
- **pandas >= 1.0.0** (Structured DataFrames for MSP files and result compilation)
- **RDKit** (Converted InChI identifiers to SMILES format when building the reference database)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Reproduce the ARUS validation result: 24 previously unannotated compounds identified from 40 expert-reviewed spectra (18 in positive mode, 6 in negative)
- Verify downloaded model/reference files with the provided sha256 checksums (e.g. b376f329f1b3759e598c9e1fd2e6b252d9f63ed2316baaff3b57be387b0b8c73 for model_positive.bin)
- Results CSV must contain exactly one row per input spectrum, with NA candidates for unmatched spectra
- On test data, correct compound should appear in the top-5 candidates in more than 76% of cases and top-ranked in over 42%
- Candidate list should be empty of compounds outside the precursor_tolerance_ppm window around the measured precursor m/z

## Limitations

- Candidate matching can only return molecules present in the 520,083-compound reference database; compounds outside it cannot be identified regardless of spectral quality
- Results obtained with a differently sized reference database are not directly comparable
- The legacy --precursor_tolerance_ppm 0 mode (3-decimal bucket comparison) fails whenever the fourth decimal digit is 5 or above — about half of all compounds — and should not be used for new work
- pip install chemembed pulls PyTorch plus CUDA libraries (~5 GB); a small /tmp may require redirecting pip's scratch space via TMPDIR

## Evidence
