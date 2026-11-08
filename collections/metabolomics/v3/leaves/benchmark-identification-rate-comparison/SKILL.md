---
name: benchmark-identification-rate-comparison
description: Use when reproducing a published metabolite-identification benchmark
  on CASMI 2016/2022 spectra, or when comparing a spectral-embedding predictor against
  SIRIUS or another baseline by ranking candidate structures per query spectrum.
when_to_use_negative:
- Do not use when the goal is a top-1 (rank-1) accuracy claim only — this skill measures
  top-5 correct-compound ranking, not top-ranked annotation rate.
- Do not compare rates computed against a differently sized reference database — the
  README states 'results obtained with a differently sized reference are not directly
  comparable', so avoid mixing reference sizes in one comparison.
- Do not apply --precursor_tolerance_ppm 0 for new work; that legacy 3-decimal bucket
  fails roughly half of compounds and is only for reproducing pre-v1.1.1 results.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: ChemEmbed
  role: 'Main pipeline: converts MSP to structured DataFrames, predicts 300-dim Mol2vec
    embeddings with a pre-trained CNN, and ranks reference compounds by cosine similarity
    under a ppm precursor filter'
  repo: https://github.com/faizanurv/ChemEmbed
- name: SIRIUS 6.0.0
  role: Baseline comparator for the head-to-head identification-rate evaluation on
    CASMI 2022
- name: RDKit
  role: Converts InChI identifiers to SMILES to build the reference database used
    for candidate matching
- name: pandas
  role: Loads the reference database (parquet/pkl) and manages the results DataFrame
    for rate computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:9a723680c2324ee5032f730470f83482b12f434c818b1aa211f04e0fb6c56113
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
  - RDKit
  - SIRIUS 6.0.0
  - pandas
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# benchmark-identification-rate-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and compare top-5 correct-compound identification rates of ChemEmbed against SIRIUS 6.0.0 on the filtered CASMI 2022 benchmark, using polarity-specific pre-trained CNN weights and cosine similarity against the 520,083-compound Mol2vec reference database.

## When to use

Use when reproducing a published metabolite-identification benchmark on CASMI 2016/2022 spectra, or when comparing a spectral-embedding predictor against SIRIUS or another baseline by ranking candidate structures per query spectrum.

## When NOT to use

- Do not use when the goal is a top-1 (rank-1) accuracy claim only — this skill measures top-5 correct-compound ranking, not top-ranked annotation rate.
- Do not compare rates computed against a differently sized reference database — the README states 'results obtained with a differently sized reference are not directly comparable', so avoid mixing reference sizes in one comparison.
- Do not apply --precursor_tolerance_ppm 0 for new work; that legacy 3-decimal bucket fails roughly half of compounds and is only for reproducing pre-v1.1.1 results.

## Inputs

- CASMI 2022 MSP files (positive and negative ionization mode spectra, no SMILES annotations)
- ChemEmbed pre-trained CNN weights model_positive.bin and model_negative.bin (1129.6 MB each)
- ChemEmbed reference database chemembed_reference_mol2vec_300d_520k.parquet or .pkl (520,083 compounds, 300-dim Mol2vec embeddings)
- YAML configuration file specifying MSP paths, model paths, reference database path, adduct polarity, and output location
- Reference compound precursor masses, SMILES/InChIKeys for identity checks

## Outputs

- CSV results file with one row per input spectrum, top 5 ranked candidates (or NA when no in-window reference exists)
- Per-mode top-5 correct-compound identification rate (fraction, positive and negative mode reported separately)
- Head-to-head comparison table of ChemEmbed vs SIRIUS 6.0.0 rates on the filtered CASMI 2022 set
- Console report of the count of spectra that went unmatched (no reference within precursor tolerance)

## How to apply

Parse the CASMI 2022 MSP files with ChemEmbed's data preprocessing (input_file_type 'without_smiles', intensity_threshold 1, tolerance 0.01, resolution 0.01, max_mz 700) to build model-ready spectra arrays. Load the polarity-matched pre-trained CNN weights — model_positive.bin for [M+H]+ and model_negative.bin for [M-H]- — and predict 300-dimensional Mol2vec embeddings from the enhanced spectra. Filter the 520,083-compound reference database to compounds whose precursor mass is within 5 ppm of the measured precursor (use 10–20 ppm for QTOF data), then rank candidates by cosine similarity and retain the top 5 per spectrum; spectra with no in-window reference are kept as rows with NA candidates. Compute the fraction of compounds whose true structure appears in the top 5, separately for positive and negative ionization mode, and compare with the reported SIRIUS rates. Note that --tolerance is a peak-binning parameter and does not affect candidate retrieval.

## Related tools

- **ChemEmbed** (Main pipeline: converts MSP to structured DataFrames, predicts 300-dim Mol2vec embeddings with a pre-trained CNN, and ranks reference compounds by cosine similarity under a ppm precursor filter) — https://github.com/faizanurv/ChemEmbed
- **SIRIUS 6.0.0** (Baseline comparator for the head-to-head identification-rate evaluation on CASMI 2022)
- **RDKit** (Converts InChI identifiers to SMILES to build the reference database used for candidate matching)
- **pandas** (Loads the reference database (parquet/pkl) and manages the results DataFrame for rate computation)

## Examples

```
chemembed --input_file_type without_smiles --adduct "+" --msp_file_positive casmi2022.msp --model_path_positive data_model_files/model_positive.bin --reference_database data_model_files/chemembed_reference_mol2vec_300d_520k.parquet --top_n_candidates 5 --prediction_results results.csv
```

## Evaluation signals

- Top-5 correct-compound rate in positive mode should reach approximately 33% and in negative mode approximately 30% (as reported for ChemEmbed on the filtered CASMI 2022 set).
- ChemEmbed rates should clearly exceed the corresponding SIRIUS rates of 28% (positive) and 17% (negative) on the same filtered compound set.
- Verify downloaded files with sha256sum (e.g. b376f329... for model_positive.bin) before running the pipeline.
- Confirm the results CSV has exactly one row per input spectrum, with NA candidates for spectra having no reference within the precursor tolerance window.
- Confirm the reference database is the full 520,083-compound Mol2vec set; any smaller reference invalidates direct comparison.

## Limitations

- Candidate matching can only return molecules present in the 520,083-compound reference database; a compound outside it cannot be identified regardless of spectral quality.
- Setting --precursor_tolerance_ppm 0 reproduces a legacy 3-decimal bucket filter that failed roughly half of compounds even when the query mass was exact, and should not be used for new work.
- --tolerance is a peak-binning parameter with no effect on candidate retrieval, so it cannot be used to widen or narrow the candidate pool.
- pip install chemembed pulls PyTorch plus CUDA libraries (~5 GB); pip's scratch space may need TMPDIR redirection on machines with a small /tmp.

## Evidence

- [results] ChemEmbed ranked the correct compound within the top five candidates in 33% of positive and 30% of negative ionization cases, whereas SIRIUS achieved 28% and 17%, respectively.: [evidence span withheld — anchor status: not_found]
- [readme] Candidate retrieval keeps every reference compound whose precursor mass is within `--precursor_tolerance_ppm` of the measured precursor.: "Candidate retrieval keeps every reference compound whose precursor mass is within `--precursor_tolerance_ppm` of the measured precursor. **This is the single most important parameter for recall.**"
- [readme] The reference database covers 520,083 compounds. Candidate matching can only return molecules that are in the database.: "The reference database covers 520,083 compounds. Candidate matching
can only return molecules that are in the database, so a compound outside it
cannot be identified regardless of spectral quality,"
- [readme] Note that `--tolerance` is a *peak binning* parameter and has no effect on candidate retrieval.: "Note that `--tolerance` is a *peak binning* parameter and has no effect on candidate
retrieval."
- [intro] The CNN is trained to predict 300-dimensional Mol2vec embeddings from these enhanced spectra.: "The CNN is trained to predict 300-dimensional Mol2vec embeddings from these enhanced spectra, and these embeddings are then compared and ranked against a reference database of millions of Mol2vec"
- [readme] Converts MSP files to structured DataFrames and preprocesses spectra data for model input.: "**Data Preprocessing:** Converts MSP files to structured DataFrames and preprocesses spectra data for model input."
- [readme] Verify the downloads before use:: "Verify the downloads before use:

```
b376f329f1b3759e598c9e1fd2e6b252d9f63ed2316baaff3b57be387b0b8c73  model_positive.bin"
