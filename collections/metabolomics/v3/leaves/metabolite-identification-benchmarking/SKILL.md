---
name: metabolite-identification-benchmarking
description: Use when evaluating a spectral-to-structure ranking model on labeled
  test spectra (MSP) against a fixed reference compound library, to reproduce or compare
  top-k annotation accuracy metrics.
when_to_use_negative:
- The goal is to compare against a differently sized or differently constructed reference
  database — candidate matching can only return molecules already in the 520,083-compound
  library, so results are not directly comparable.
- Input spectra are unlabeled or from a real discovery run where no query SMILES exists
  to compute Tanimoto similarity against the top-ranked candidates.
- The task is to retrain or fine-tune the CNN on new spectra rather than to evaluate
  the released pretrained model on a fixed benchmark.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ChemEmbed
  role: Provides the pretrained CNN that predicts 300-dimensional Mol2vec embeddings
    from enhanced (merged, binarized, peak-binned) MS/MS spectra and the CLI to run
    candidate retrieval.
  repo: https://github.com/faizanurv/ChemEmbed
- name: RDKit
  role: Used to compute Tanimoto similarity between the query SMILES and each ranked
    candidate SMILES for top-1 / top-5 correctness scoring.
  repo: https://github.com/rdkit/rdkit
- name: pandas >= 1.0.0
  role: Reads the reference database (parquet or pickle) and writes the prediction_results.csv
    with ranked candidates and summary metrics.
  repo: https://pandas.pydata.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:d9802fb1ba11af71c6863c904bd70542bae48d8d4856c99cbe993b4206d77696
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metabolite-identification-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a metabolite-identification model (e.g. ChemEmbed CNN) on a held-out test split of MS/MS spectra by ranking candidate compounds from a 520,083-compound Mol2vec reference database and reporting top-1 (Tanimoto >= 0.95) and top-5 annotation rates.

## When to use

Use when evaluating a spectral-to-structure ranking model on labeled test spectra (MSP) against a fixed reference compound library, to reproduce or compare top-k annotation accuracy metrics.

## When NOT to use

- The goal is to compare against a differently sized or differently constructed reference database — candidate matching can only return molecules already in the 520,083-compound library, so results are not directly comparable.
- Input spectra are unlabeled or from a real discovery run where no query SMILES exists to compute Tanimoto similarity against the top-ranked candidates.
- The task is to retrain or fine-tune the CNN on new spectra rather than to evaluate the released pretrained model on a fixed benchmark.

## Inputs

- Held-out test-split spectra in MSP file format (e.g. msp_file_positive or msp_file_negative)
- Pretrained CNN model checkpoint (model_positive.bin or model_negative.bin, ~1129.6 MB each)
- Reference database of 520,083 compounds with 300-dim Mol2vec embeddings (chemembed_reference_mol2vec_300d_520k.parquet or .pkl)
- Measured precursor m/z values for each query spectrum

## Outputs

- prediction_results.csv containing the ranked top-5 candidate list per spectrum
- Summary of top-1 (Tanimoto >= 0.95) and top-5 annotation rates
- Count of spectra with no reference candidate inside the precursor window (reported as NA)

## How to apply

Load the held-out test-split spectra from the MSP file and select the polarity-specific pretrained CNN checkpoint (model_positive.bin or model_negative.bin) together with the 520,083-compound reference database released with the package. Preprocess each spectrum by merging MS2 scans, binarizing intensities with the intensity threshold, and applying peak binning (--tolerance) so spectra are in model-input format. Run batched CNN inference (batch_size 32, GPU if available) to predict 300-dimensional Mol2vec embeddings, then match each embedding to the reference by cosine similarity, restricting candidates to compounds whose precursor mass falls within --precursor_tolerance_ppm (5 ppm default) of the measured precursor. Compute per-spectrum Tanimoto similarity between query and candidate SMILES with RDKit, count a spectrum as top-1 correct if the highest-ranked candidate has Tanimoto >= 0.95 and top-5 correct if any of the top five candidates does, then aggregate the fractions. The acceptance criterion from the article is that top-1 hits exceed 42% and top-5 hits exceed 76% on the test dataset.

## Related tools

- **ChemEmbed** (Provides the pretrained CNN that predicts 300-dimensional Mol2vec embeddings from enhanced (merged, binarized, peak-binned) MS/MS spectra and the CLI to run candidate retrieval.) — https://github.com/faizanurv/ChemEmbed
- **RDKit** (Used to compute Tanimoto similarity between the query SMILES and each ranked candidate SMILES for top-1 / top-5 correctness scoring.) — https://github.com/rdkit/rdkit
- **pandas >= 1.0.0** (Reads the reference database (parquet or pickle) and writes the prediction_results.csv with ranked candidates and summary metrics.) — https://pandas.pydata.org

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- The reproduced top-1 correct rate (Tanimoto >= 0.95) exceeds 42% and the top-5 rate exceeds 76% on the test dataset, matching the article's abstract findings.
- The results file contains exactly one row per input spectrum, including rows with NA candidates when no reference compound falls inside the precursor tolerance window.
- Downloads verify against the published sha256 sums (e.g. 41a88ef1c84e3f0bec2babfb6cac8941ae4fc37168622a94676a3e9c6c753df0 for the parquet reference).
- All returned candidates fall within the precursor_tolerance_ppm window of the measured precursor (default 5 ppm for high-resolution data).
- The number of unmatched spectra printed by the run is consistent with the number of NA rows in prediction_results.csv.

## Limitations

- The reference database covers 520,083 compounds; a compound outside it cannot be identified regardless of spectral quality, and results obtained with a differently sized reference are not directly comparable.
- Setting --precursor_tolerance_ppm to 0 restores a legacy 3-decimal bucket filter that failed for roughly half of all compounds and should not be used for new work.
- --tolerance is a peak binning parameter and has no effect on candidate retrieval.
- pip install chemembed pulls in PyTorch with CUDA libraries (~5 GB); the models and reference database are downloaded separately from the v1.0.1 release.

## Evidence
