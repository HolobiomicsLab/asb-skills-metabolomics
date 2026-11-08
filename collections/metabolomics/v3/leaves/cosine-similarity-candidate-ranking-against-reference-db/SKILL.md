---
name: cosine-similarity-candidate-ranking-against-reference-db
description: Use when you have CNN-predicted molecular embeddings from MS/MS spectra
  and a reference database of Mol2vec embeddings, and need to retrieve and rank top
  candidate compounds per spectrum by cosine similarity.
when_to_use_negative:
- The compound of interest is not in the reference database — candidate matching can
  only return molecules that are in the database, so a compound outside it cannot
  be identified regardless of spectral quality.
- You need to compare full spectra directly rather than embeddings — this skill operates
  on 300-dim Mol2vec embeddings, not raw or merged spectra as similarity inputs.
- Results must be comparable to a run with a differently sized reference database
  — results obtained with a differently sized reference are not directly comparable.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemEmbed
  role: Predicts the 300-dimensional Mol2vec embedding from merged MS/MS spectra;
    its CLI performs cosine-similarity matching against the reference database with
    precursor-ppm filtering
  repo: https://github.com/faizanurv/ChemEmbed.git
- name: FAISS
  role: Single FAISS index searched once per sample for fast cosine-similarity candidate
    retrieval, with the precursor m/z post-filter applied to the results
- name: pandas >= 1.0.0
  role: Loads the reference database (parquet/pickle) and structures the per-spectrum
    candidate results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:c3e163e0befbd5651aec71ff5be83ba90a106fd9a063edbc8d2a78fe2ccb4a0d
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
  - FAISS
  - Python+
  - RDKit
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cosine-similarity-candidate-ranking-against-reference-db

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank candidate metabolites for an MS/MS spectrum by comparing the 300-dimensional Mol2vec embedding predicted by ChemEmbed's CNN against a precomputed reference database using cosine similarity, restricted by precursor mass tolerance. This is the core candidate-retrieval and ranking step of ChemEmbed's metabolite identification pipeline.

## When to use

Use when you have CNN-predicted molecular embeddings from MS/MS spectra and a reference database of Mol2vec embeddings, and need to retrieve and rank top candidate compounds per spectrum by cosine similarity.

## When NOT to use

- The compound of interest is not in the reference database — candidate matching can only return molecules that are in the database, so a compound outside it cannot be identified regardless of spectral quality.
- You need to compare full spectra directly rather than embeddings — this skill operates on 300-dim Mol2vec embeddings, not raw or merged spectra as similarity inputs.
- Results must be comparable to a run with a differently sized reference database — results obtained with a differently sized reference are not directly comparable.

## Inputs

- 300-dimensional Mol2vec embeddings predicted by the ChemEmbed CNN from merged, binarized MS/MS spectra
- Reference database of Mol2vec embeddings with SMILES, InChIKeys and precursor masses (chemembed_reference_mol2vec_300d_520k.parquet or .pkl, 520,083 compounds)
- Measured precursor m/z per query spectrum
- Precursor mass tolerance in ppm (--precursor_tolerance_ppm)

## Outputs

- Ranked top-N candidate molecules per spectrum (top_n_candidates, e.g. 5) by cosine similarity
- Prediction results file (e.g. results.csv / prediction_results.csv) with one row per input spectrum, NA candidates for unmatched spectra
- Count of spectra with no reference candidate inside the precursor window

## How to apply

Load the reference database (e.g. chemembed_reference_mol2vec_300d_520k.parquet with 520,083 compounds, 300-dim Mol2vec embeddings) once before the sample loop rather than once per sample. For each spectrum, run batched CNN inference (batch_size 32, GPU if available) to predict the 300-dimensional embedding, then match predicted embeddings against the reference database to find top candidate molecules based on cosine similarity — in practice one FAISS index and a single search per sample with the precursor m/z post-filter applied to the FAISS results. Restrict candidates to reference compounds whose precursor mass is within --precursor_tolerance_ppm of the measured precursor (5 ppm default for Orbitrap, 10-20 for QTOF, 0 to restore the legacy exact 3-decimal bucket). Keep the top-N candidates (e.g. --top_n_candidates 5) and report spectra with no reference inside the window as NA candidates so the results file always has one row per input spectrum.

## Related tools

- **ChemEmbed** (Predicts the 300-dimensional Mol2vec embedding from merged MS/MS spectra; its CLI performs cosine-similarity matching against the reference database with precursor-ppm filtering) — https://github.com/faizanurv/ChemEmbed.git
- **FAISS** (Single FAISS index searched once per sample for fast cosine-similarity candidate retrieval, with the precursor m/z post-filter applied to the results)
- **pandas >= 1.0.0** (Loads the reference database (parquet/pickle) and structures the per-spectrum candidate results)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive your_spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- The results file contains exactly one row per input spectrum, with NA candidates (not omission) for spectra with no reference inside the precursor window.
- Every retained candidate's precursor mass falls within --precursor_tolerance_ppm of the measured precursor (5 ppm default).
- The run prints how many spectra went unmatched, and this count agrees with the number of NA rows in the output.
- On the test dataset, top-1 hits (Tanimoto >= 0.95) exceed 42% and top-5 hits exceed 76%, matching the reported ranking results.
- The reference database loaded contains 520,083 compounds with 300-dimensional embeddings, matching the released parquet/pickle.

## Limitations

- Candidate matching can only return molecules that are in the reference database; a compound outside the 520,083-compound database cannot be identified regardless of spectral quality.
- Setting --precursor_tolerance_ppm 0 restores the legacy exact 3-decimal bucket filter, which failed whenever the fourth decimal digit was 5 or above — about half of all compounds, even when the query mass was exact — and should not be used for new work.
- --tolerance is a peak binning parameter and has no effect on candidate retrieval.
- Results obtained with a differently sized reference database are not directly comparable.

## Evidence
