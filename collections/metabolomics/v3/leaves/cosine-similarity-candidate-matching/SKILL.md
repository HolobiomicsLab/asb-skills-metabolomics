---
name: cosine-similarity-candidate-matching
description: Use when a predicted embedding from a spectrum (e.g. ChemEmbed CNN output,
  positive or negative mode) must be matched to candidate structures in the 520,083-compound
  Mol2vec reference database using precursor m/z filtering plus cosine-similarity
  ranking.
when_to_use_negative:
- 'Using --tolerance as a candidate-retrieval filter: the README states it is a peak
  binning parameter with no effect on candidate retrieval.'
- Comparing results against benchmarks computed with a differently sized reference
  database — such results are not directly comparable.
- Attempting to identify compounds outside the 520,083-compound reference database;
  candidate matching can only return molecules that are in the database.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemEmbed
  role: Provides the CLI pipeline (chemembed command) that performs precursor-tolerance
    filtering and cosine-similarity candidate matching against the reference database
  repo: https://github.com/faizanurv/ChemEmbed
- name: Mol2vec
  role: Source of the 300-dimensional structural embeddings used for both predicted
    and reference representations being compared by cosine similarity
- name: FAISS
  role: Optional one-index search backend; a single GPU call retrieves all of a sample's
    query embeddings, followed by a precursor m/z post-filter
- name: pandas >= 1.0.0
  role: Loads the reference database (read_pickle or read_parquet via pyarrow) and
    structures per-spectrum candidate results
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:e12493d2b73c9c055bf0e9627d1e7ef14e891386948203daff5346d9257ced11
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
  - PyTorch
  - Python+
  - RDKit
  - numpy >=
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# cosine-similarity-candidate-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks molecular candidates for MS/MS spectra by cosine similarity between CNN-predicted 300-dimensional Mol2vec embeddings and a 520,083-compound Mol2vec reference database, after filtering reference compounds by precursor mass tolerance (5 ppm default).

## When to use

Use when a predicted embedding from a spectrum (e.g. ChemEmbed CNN output, positive or negative mode) must be matched to candidate structures in the 520,083-compound Mol2vec reference database using precursor m/z filtering plus cosine-similarity ranking.

## When NOT to use

- Using --tolerance as a candidate-retrieval filter: the README states it is a peak binning parameter with no effect on candidate retrieval.
- Comparing results against benchmarks computed with a differently sized reference database — such results are not directly comparable.
- Attempting to identify compounds outside the 520,083-compound reference database; candidate matching can only return molecules that are in the database.

## Inputs

- Predicted 300-dimensional Mol2vec embeddings per spectrum from the pre-trained CNN (model_positive.bin for [M+H]+ or model_negative.bin for [M-H]-)
- Reference database file: chemembed_reference_mol2vec_300d_520k.parquet (or .pkl) with 520,083 compounds, 300-dim mol2vec embeddings, SMILES, InChIKeys and precursor masses
- Measured precursor m/z per spectrum (from MSP/MGF input)
- --precursor_tolerance_ppm parameter (5 default; 10-20 for QTOF; 0 for legacy exact 3-decimal bucket)
- --top_n_candidates parameter (5 in CASMI 2022 and ARUS workflows)

## Outputs

- Ranked top-N candidate molecule list per spectrum (e.g. prediction_results CSV), one row per input spectrum
- NA candidate entries for spectra with no reference compound inside the precursor window, plus a printed count of unmatched spectra
- Per-polarity top-5 correct-compound fractions (e.g. 33% positive / 30% negative on filtered CASMI 2022)

## How to apply

Load the reference database (chemembed_reference_mol2vec_300d_520k.parquet or .pkl) once before the sample loop, keeping all 520,083 compounds with their 300-dim embeddings, SMILES, InChIKeys and precursor masses. For each spectrum, retain every reference compound whose precursor mass is within --precursor_tolerance_ppm of the measured precursor (5 ppm default for Orbitrap/high-resolution data; 10-20 for QTOF), then rank survivors by cosine similarity between predicted and reference embeddings and keep the top N candidates (--top_n_candidates 5 in the benchmarks). Spectra with no in-window reference are reported with NA candidates rather than omitted, so the results file always has one row per input spectrum, and the run prints the unmatched count. When a FAISS index is used, apply the precursor m/z post-filter to the FAISS results to preserve ChemEmbed's original matching semantics; note that --tolerance is only a peak-binning parameter and has no effect on candidate retrieval.

## Related tools

- **ChemEmbed** (Provides the CLI pipeline (chemembed command) that performs precursor-tolerance filtering and cosine-similarity candidate matching against the reference database) — https://github.com/faizanurv/ChemEmbed
- **Mol2vec** (Source of the 300-dimensional structural embeddings used for both predicted and reference representations being compared by cosine similarity)
- **FAISS** (Optional one-index search backend; a single GPU call retrieves all of a sample's query embeddings, followed by a precursor m/z post-filter)
- **pandas >= 1.0.0** (Loads the reference database (read_pickle or read_parquet via pyarrow) and structures per-spectrum candidate results)

## Examples

```
chemembed --adduct "+" --msp_file_positive spectra.msp --model_path_positive data_model_files/model_positive.bin --reference_database chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- On the filtered CASMI 2022 benchmark, top-5 correct-compound rates reach or approach 33% (positive mode, 107 compounds) and 30% (negative mode, 64 compounds), clearly above SIRIUS 6.0.0's 28% and 17%
- On the test split, the matching pipeline supports top-1 annotation in over 42% of cases and top-5 in more than 76% of cases
- Results file contains exactly one row per input spectrum; spectra with no in-window reference carry NA candidates and the unmatched count is printed
- Reference database integrity: 520,083 compounds; verify the downloaded file sha256 (e.g. 41a88ef1c84e3f0bec2babfb6cac8941ae4fc37168622a94676a3e9c6c753df0 for the parquet) and confirm a differently sized reference was not used
- Precursor filter behaviour matches the configured --precursor_tolerance_ppm (5 ppm default), not the legacy 3-decimal bucket unless explicitly requested

## Limitations

- Candidate matching can only return molecules present in the 520,083-compound reference database; compounds outside it cannot be identified regardless of spectral quality
- The legacy --precursor_tolerance_ppm 0 setting (exact 3-decimal bucket) failed whenever the fourth decimal digit was 5 or above — about half of all compounds even with exact query masses — and should not be used for new work
- The 5 ppm default assumes high-resolution instruments (Orbitrap); QTOF data may require 10-20 ppm
- The .pkl reference is riskier than the parquet because pandas.read_pickle executes code while unpickling; the parquet requires installing pyarrow separately

## Evidence
