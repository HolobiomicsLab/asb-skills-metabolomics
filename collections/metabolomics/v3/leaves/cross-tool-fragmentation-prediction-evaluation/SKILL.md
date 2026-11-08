---
name: cross-tool-fragmentation-prediction-evaluation
description: Use when comparing MS/MS spectral prediction quality across in silico
  fragmentation tools (e.g., FIORA vs ICEBERG vs CFM-ID) and assessing generalization
  to compounds structurally dissimilar from the training set via sqrt-weighted cosine
  similarity stratified by maximum Tanimoto similarity.
when_to_use_negative:
- No experimentally measured reference spectra exist for the test compounds — the
  sqrt-weighted cosine metric requires scoring predictions against reference spectra.
- The goal is retention time (RT) or collision cross section (CCS) prediction rather
  than MS/MS spectral quality benchmarking.
- Only a single tool's predictions are being validated without comparison baselines
  — this skill is specifically a cross-tool evaluation stratified by training-set
  similarity.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0622
tools:
- name: FIORA
  role: Primary spectral predictor under test; its fiora-predict CLI generates predictions
    and fiora-eval computes the spectral_sqrt_cosine scoring metric
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: State-of-the-art baseline fragmentation predictor, run from the ms-pred repository
    with test compounds removed from its training data
  repo: https://github.com/coleygroup/ms-pred
- name: CFM-ID
  role: Baseline fragmentation predictor (METLIN17-trained, not retrained), run via
    Docker and evaluated at the closest matching collision energy
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: RDKit
  role: Computes Morgan fingerprints (2048 bits, radius 3) used to calculate maximum
    Tanimoto similarity between test and training compounds
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:720527fb5ec1d3dadc841ff4122607428a58b679bf5526b2b3fffea16160957d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-tool-fragmentation-prediction-evaluation

## Summary

Benchmark in silico fragmentation tools (FIORA, ICEBERG, CFM-ID) by scoring predicted MS/MS spectra against reference spectra with the sqrt-weighted cosine similarity, then stratify performance by each test compound's maximum Tanimoto similarity to the training set. This is how the article demonstrates FIORA's robustness on structurally dissimilar compounds (median cosine 0.63 in the 0.2–0.3 Tanimoto bin) while outperforming ICEBERG and CFM-ID.

## When to use

Use when comparing MS/MS spectral prediction quality across in silico fragmentation tools (e.g., FIORA vs ICEBERG vs CFM-ID) and assessing generalization to compounds structurally dissimilar from the training set via sqrt-weighted cosine similarity stratified by maximum Tanimoto similarity.

## When NOT to use

- No experimentally measured reference spectra exist for the test compounds — the sqrt-weighted cosine metric requires scoring predictions against reference spectra.
- The goal is retention time (RT) or collision cross section (CCS) prediction rather than MS/MS spectral quality benchmarking.
- Only a single tool's predictions are being validated without comparison baselines — this skill is specifically a cross-tool evaluation stratified by training-set similarity.

## Inputs

- MSnLib spectral library (resources/data/msnlib/library.csv) with SMILES, precursor types, and collision energies; positive-mode [M+H]+ test and training splits
- Reference MS/MS spectra for each test compound (mgf/msp)
- Trained FIORA model checkpoint (checkpoints/fiora.pt)
- Pretrained ICEBERG model weights (from coleygroup/ms-pred)
- CFM-ID Docker container (wishartlab/cfmid)

## Outputs

- Predicted MS/MS spectra in mgf or msp format for FIORA, ICEBERG, and CFM-ID
- Per-compound sqrt-weighted cosine similarity scores against reference spectra
- tanimoto_bin_summary.csv (per-Tanimoto-bin median cosine similarity and IQR per tool)
- fig3_tanimoto_cosine_similarity.png (binned median cosine similarity plot with IQR bands)

## How to apply

Load resources/data/msnlib/library.csv (MSnLib) and split positive-mode [M+H]+ compounds into training and test sets, removing all test compounds from the training data for FIORA and ICEBERG to preserve test integrity. Compute Morgan fingerprints (2048 bits, radius 3) with RDKit and, for each test compound, its maximum Tanimoto similarity against all training compounds. Predict MS/MS spectra for every test compound with FIORA (fiora-predict using the checkpoints/fiora.pt model), ICEBERG, and CFM-ID under matching collision energies and instrument type (HCD), evaluating CFM-ID at the closest matching collision energy among 10, 20, 40 eV. Score each predicted spectrum against its reference spectrum with the sqrt-weighted cosine similarity (spectral_sqrt_cosine, FIORA's default evaluation metric via fiora-eval). Bin compounds into fixed maximum-Tanimoto similarity intervals and compute per-bin median cosine similarity and interquartile range for each tool. Validate that FIORA's median cosine in the 0.2–0.3 Tanimoto bin reproduces ~0.63, performance stays ≥0.8 for Tanimoto 0.6–1, and FIORA plots above ICEBERG and CFM-ID across bins.

## Related tools

- **FIORA** (Primary spectral predictor under test; its fiora-predict CLI generates predictions and fiora-eval computes the spectral_sqrt_cosine scoring metric) — https://github.com/BAMeScience/fiora
- **ICEBERG** (State-of-the-art baseline fragmentation predictor, run from the ms-pred repository with test compounds removed from its training data) — https://github.com/coleygroup/ms-pred
- **CFM-ID** (Baseline fragmentation predictor (METLIN17-trained, not retrained), run via Docker and evaluated at the closest matching collision energy) — https://hub.docker.com/r/wishartlab/cfmid
- **RDKit** (Computes Morgan fingerprints (2048 bits, radius 3) used to calculate maximum Tanimoto similarity between test and training compounds)

## Examples

```
fiora-predict -i test_compounds.csv -o predicted_spectra.mgf --model checkpoints/fiora.pt
```

## Evaluation signals

- FIORA's median cosine similarity in the 0.2–0.3 maximum-Tanimoto bin reproduces the reported value of 0.63.
- FIORA's per-bin median cosine similarity exceeds ICEBERG and CFM-ID across all Tanimoto bins.
- Median cosine similarity remains stable at ≥0.8 for Tanimoto similarity 0.6–1.
- Scores are computed with spectral_sqrt_cosine (FIORA's default evaluation metric via fiora-eval), not unweighted cosine.
- All test-set compounds verified as removed from FIORA and ICEBERG training data before scoring.

## Limitations

- CFM-ID was trained on METLIN17, so train/test compound separation is not guaranteed for CFM-ID comparisons; retraining CFM-ID with the authors' dataset was computationally infeasible.
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by lack of experimental metadata, especially collision energies.
- Single-step fragmentation is FIORA's biggest limitation; multi-step fragmentation is not implemented.
- The paper used MSnLib v1.0 (Zenodo 11163381), while the repository's downloader defaults to MSnLib v7 (Zenodo 16984129), so reproduced numbers may shift.

## Evidence

- [abstract] FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality: "FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality"
- [intro] We benchmark the performance against the top-performing methods, CFM-ID and ICEBERG.: "We benchmark the performance against the top-performing methods, CFM-ID and ICEBERG."
- [discussion] To preserve the integrity of the test results, all compounds from the test sets were removed from the training data for FIORA and ICEBERG.: "To preserve the integrity of the test results, all compounds from the test sets were removed from the training data for FIORA and ICEBERG."
- [discussion] CFM-ID was trained on the METLIN17 library, which means that this separation is not guaranteed.: "CFM-ID was trained on the METLIN17 library, which means that this separation is not guaranteed."
- [readme] Scripts for downloading and preprocessing MSnLib are provided in `resources/data/msnlib` (`download_msnlib.py` and `preprocess_msnlib.py`).: "Scripts for downloading and preprocessing MSnLib are provided in `resources/data/msnlib` (`download_msnlib.py` and `preprocess_msnlib.py`)."
