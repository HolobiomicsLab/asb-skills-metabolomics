---
name: spectral-prediction-benchmark-visualization
description: Use when comparing multiple in silico MS/MS spectral prediction tools
  on a common test split and you must show how prediction quality degrades as test
  compounds become structurally dissimilar from training compounds.
when_to_use_negative:
- The comparison is on training-set-adjacent compounds only, not a generalizability/structural-novelty
  analysis — a simple overall cosine similarity table suffices
- The reference spectra lack annotated collision energies, so FIORA/ICEBERG/CFM-ID
  cannot be run under matched collision energy and instrument type
- The goal is compound identification (candidate ranking), not spectral prediction
  quality assessment
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Primary spectral predictor under benchmark; also supplies fiora-eval with
    the spectral_sqrt_cosine metric used for scoring
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: Baseline spectral prediction method from the coleygroup/ms-pred repository,
    trained after removing all test compounds from its training data
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Baseline spectral prediction method, run via its Docker container; trained
    on METLIN17 so train/test separation is not guaranteed
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: MSnLib
  role: Source of the multi-stage fragmentation spectra (library.csv) providing reference
    spectra, SMILES, precursor types, and collision energies
  repo: https://zenodo.org/records/16984129
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:3a7f788e22f0d65881891cde50d55126d124982b587186f2d67deabd83d806f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  - MSnLib
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# spectral-prediction-benchmark-visualization

## Summary

Benchmarks predicted MS/MS spectra (FIORA vs. ICEBERG vs. CFM-ID) against reference spectra using sqrt-weighted cosine similarity, stratified by structural novelty (maximum Tanimoto similarity of each test compound to the training set), and renders the binned median similarity plot with IQR bands as in Fig. 3 of the FIORA paper.

## When to use

Use when comparing multiple in silico MS/MS spectral prediction tools on a common test split and you must show how prediction quality degrades as test compounds become structurally dissimilar from training compounds.

## When NOT to use

- The comparison is on training-set-adjacent compounds only, not a generalizability/structural-novelty analysis — a simple overall cosine similarity table suffices
- The reference spectra lack annotated collision energies, so FIORA/ICEBERG/CFM-ID cannot be run under matched collision energy and instrument type
- The goal is compound identification (candidate ranking), not spectral prediction quality assessment

## Inputs

- Preprocessed MSnLib spectral library CSV (resources/data/msnlib/library.csv) with SMILES, precursor types, and collision energies for [M+H]+ training/test splits
- Reference MS/MS spectra for the test compounds (mgf/msp)
- Predicted spectra from FIORA (fiora-predict, checkpoints/fiora.pt), ICEBERG (coleygroup/ms-pred), and CFM-ID (Docker: wishartlab/cfmid) at matched collision energies and instrument type
- RDKit Morgan fingerprints (2048 bits, radius 3) of test and training compounds

## Outputs

- Per-bin summary CSV of median cosine similarity and IQR per tool (tanimoto_bin_summary.csv)
- Binned median cosine similarity plot with IQR bands (fig3_tanimoto_cosine_similarity.png)
- Per-compound spectral_sqrt_cosine similarity scores for FIORA, ICEBERG, and CFM-ID

## How to apply

Load the preprocessed MSnLib spectral library (resources/data/msnlib/library.csv) and split into positive-mode [M+H]+ training and test compounds. Compute Morgan fingerprints (2048 bits, radius 3) with RDKit and, for each test compound, its maximum Tanimoto similarity against all training compounds. Predict MS/MS spectra for every test compound with fiora-predict (checkpoints/fiora.pt), ICEBERG, and CFM-ID under matching collision energies and instrument type, then score each prediction against its reference spectrum with the sqrt-weighted cosine metric (spectral_sqrt_cosine via fiora-eval). Bin compounds into fixed Tanimoto intervals and compute the per-bin median cosine similarity and interquartile range per tool, then render the median-vs-Tanimoto plot with IQR bands (fig3_tanimoto_cosine_similarity.png, tanimoto_bin_summary.csv). The rationale is to test generalizability: FIORA should stay above ICEBERG and CFM-ID in every bin and remain robust even for the most dissimilar compounds.

## Related tools

- **FIORA** (Primary spectral predictor under benchmark; also supplies fiora-eval with the spectral_sqrt_cosine metric used for scoring) — https://github.com/BAMeScience/fiora
- **ICEBERG** (Baseline spectral prediction method from the coleygroup/ms-pred repository, trained after removing all test compounds from its training data) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Baseline spectral prediction method, run via its Docker container; trained on METLIN17 so train/test separation is not guaranteed) — https://hub.docker.com/r/wishartlab/cfmid
- **MSnLib** (Source of the multi-stage fragmentation spectra (library.csv) providing reference spectra, SMILES, precursor types, and collision energies) — https://zenodo.org/records/16984129

## Examples

```
fiora-predict -i test_compounds.csv -o fiora_predictions.mgf --model checkpoints/fiora.pt
```

## Evaluation signals

- Median FIORA cosine similarity in the 0.2–0.3 maximum-Tanimoto bin reproduces the reported value of 0.63
- FIORA's per-bin median cosine similarity is ≥0.8 for Tanimoto 0.6–1 and lies above ICEBERG and CFM-ID across all bins
- Every test compound has a maximum-Tanimoto value computed against the full training split (Morgan 2048-bit, radius 3 fingerprints) and falls into exactly one bin
- All three tools are evaluated on the same test compounds under matching collision energies and instrument type; each prediction-vs-reference pair yields a valid spectral_sqrt_cosine score
- Outputs include both the rendered figure (fig3_tanimoto_cosine_similarity.png) and its underlying per-bin statistics (tanimoto_bin_summary.csv) that agree with each other

## Limitations

- CFM-ID was trained on METLIN17, so train/test compound separation is not guaranteed for CFM-ID, biasing its curve upward relative to fully held-out tools
- Retraining CFM-ID with the authors' dataset was found computationally infeasible, so cross-tool comparisons partially reflect different training data
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limiting coverage of chemical space and hence performance in low-Tanimoto bins

## Evidence

- [abstract] FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality: "FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality"
- [discussion] test compounds removed from training: "To preserve the integrity of the test results, all compounds from the test sets were removed from the training data for FIORA and ICEBERG."
- [discussion] CFM-ID trained on METLIN17: "CFM-ID was trained on the METLIN17 library, which means that this separation is not guaranteed."
- [intro] generalizability benchmark objective: "We benchmark the performance against the top-performing methods, CFM-ID and ICEBERG."
- [readme] fiora-predict usage: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
