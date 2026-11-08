---
name: topk-annotation-rate-evaluation
description: Use when a trained CNN predicts molecular embeddings from MS/MS spectra
  and you must quantify annotation accuracy as top-1 and top-5 correctness against
  a reference database of 520,083 compounds.
when_to_use_negative:
- The compound under study is not present in the 520,083-compound reference database
  — candidate matching cannot return molecules outside it, so top-k rates would be
  meaningless.
- You only need raw embedding quality (cosine similarity / Euclidean distance) rather
  than end-to-end annotation ranking — a direct embedding-distance metric is more
  appropriate.
- The query precursor mass tolerance is set to 0 (the legacy three-decimal bucket)
  for new work — the README states this comparison fails for about half of all compounds
  even when the query mass is exact.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ChemEmbed
  role: 'Executes the full annotation pipeline: CNN embedding prediction, precursor-ppm
    filtered cosine-similarity ranking, and top-k candidate reporting via chemembed
    CLI with --top_n_candidates and --prediction_results'
  repo: github.com/faizanurv/ChemEmbed
- name: pandas >= 1.0.0
  role: Loads the reference database (parquet or pickle) and structured spectrum DataFrames
    for matching and per-spectrum result assembly
- name: PyTorch
  role: Runs batched CNN inference (batch_size=32, GPU) to generate 300-dimensional
    Mol2vec embeddings from preprocessed spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:3690c3ba5c92ff4494ec58be36880002c50bb857a87f896dbea2541bceade956
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
  - PyTorch
  - RDKit
  - numpy >=
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# topk-annotation-rate-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluation procedure for ChemEmbed-style metabolite annotation that ranks candidate molecules by cosine similarity between predicted and reference 300-dim Mol2vec embeddings and reports top-1 and top-5 correct-annotation rates per spectrum.

## When to use

Use when a trained CNN predicts molecular embeddings from MS/MS spectra and you must quantify annotation accuracy as top-1 and top-5 correctness against a reference database of 520,083 compounds.

## When NOT to use

- The compound under study is not present in the 520,083-compound reference database — candidate matching cannot return molecules outside it, so top-k rates would be meaningless.
- You only need raw embedding quality (cosine similarity / Euclidean distance) rather than end-to-end annotation ranking — a direct embedding-distance metric is more appropriate.
- The query precursor mass tolerance is set to 0 (the legacy three-decimal bucket) for new work — the README states this comparison fails for about half of all compounds even when the query mass is exact.

## Inputs

- Preprocessed MSP spectra (positive [M+H]+ or negative [M-H]- mode) as structured DataFrames
- Pre-trained CNN model file (model_positive.bin or model_negative.bin)
- Reference database of 520,083 compounds with 300-dim Mol2vec embeddings (chemembed_reference_mol2vec_300d_520k.parquet or .pkl)
- Measured precursor m/z per spectrum for ppm tolerance filtering

## Outputs

- results.csv with top N candidates per spectrum (top_n_candidates=5), one row per input spectrum, NA candidates where no reference falls within the precursor window
- Per-spectrum top-1 and top-5 correctness counts
- Summary annotation rates (fraction of spectra with correct compound at rank 1 and within rank 5)

## How to apply

Run batched CNN inference (batch_size=32) on preprocessed spectra to produce 300-dimensional Mol2vec embeddings, then match each predicted embedding to a reference database (chemembed_reference_mol2vec_300d_520k.parquet or .pkl) by cosine similarity, keeping only reference compounds whose precursor mass is within --precursor_tolerance_ppm of the measured precursor (5 ppm for Orbitrap; 10–20 ppm for QTOF). Rank surviving candidates by cosine similarity and record, per spectrum, whether the true compound appears at rank 1 and within rank 5. Spectra with no reference inside the mass window must be reported with NA candidates rather than omitted, so the results file has one row per input spectrum and the run prints how many spectra went unmatched. On the test dataset, accept the evaluation if top-ranked candidate annotations exceed 42% of cases and top-five annotations exceed 76%; a merged- versus individual-spectrum ablation is additionally validated by mean cosine similarity rising from 0.93 to 0.97 and mean Euclidean distance falling from 40 to 23.1 in positive mode.

## Related tools

- **ChemEmbed** (Executes the full annotation pipeline: CNN embedding prediction, precursor-ppm filtered cosine-similarity ranking, and top-k candidate reporting via chemembed CLI with --top_n_candidates and --prediction_results) — github.com/faizanurv/ChemEmbed
- **pandas >= 1.0.0** (Loads the reference database (parquet or pickle) and structured spectrum DataFrames for matching and per-spectrum result assembly)
- **PyTorch** (Runs batched CNN inference (batch_size=32, GPU) to generate 300-dimensional Mol2vec embeddings from preprocessed spectra)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Top-ranked candidate annotations exceed 42% of cases on the test dataset
- Correct compound appears within the top five for more than 76% of cases on the test dataset
- Results file contains exactly one row per input spectrum, with NA candidates reported for unmatched precursors rather than omitted rows
- In a merged-vs-individual-spectrum ablation, mean cosine similarity rises from 0.93 to 0.97 and mean Euclidean distance falls from 40 to 23.1 in positive ionization
- SHA256 of the downloaded reference database matches the published checksum (41a88ef1c84e3f0bec2babfb6cac8941ae4fc37168622a94676a3e9c6c753df0 for the parquet)

## Limitations

- Candidate matching can only return molecules present in the 520,083-compound reference database; compounds outside it cannot be identified regardless of spectral quality.
- Results obtained with a differently sized reference database are not directly comparable.
- The legacy precursor tolerance of 0 (three-decimal bucket) truncates reference masses while truncating query masses, causing failures for roughly half of compounds even with exact query masses; it is retained only for reproducing earlier results.
- --tolerance is a peak-binning parameter and has no effect on candidate retrieval.

## Evidence
