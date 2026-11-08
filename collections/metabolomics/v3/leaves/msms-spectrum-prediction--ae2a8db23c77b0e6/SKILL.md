---
name: msms-spectrum-prediction
description: Use when you need in silico MS/MS spectra for compounds with known SMILES
  (e.g., CASMI challenge sets, [M+H]+/[M-H]- precursors) and must quantify prediction
  quality via cosine similarity versus annotated peak-intensity coverage.
when_to_use_negative:
- Compounds require multi-step (MSn) fragmentation prediction — the article states
  'the lack of multi-step fragmentation is currently the biggest limitation' of FIORA
- Predictions need RT or CCS values tuned for production use — RT/CCS prediction is
  a prototype stage with untuned hyperparameters
- Spectra have missing collision energies or unclear formats — these were explicitly
  excluded from FIORA's training/evaluation data
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Open-source GNN used via the fiora-predict CLI to simulate tandem mass spectra,
    including fragment ion probabilities and intensities, from SMILES inputs
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: State-of-the-art baseline fragmentation predictor (via coleygroup/ms-pred)
    that FIORA is benchmarked against for prediction quality
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Baseline method evaluated at the closest matching collision energy among 10,
    20, 40 eV for comparison with FIORA predictions
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:deb9974dc70518dd93fdb1beeef25b8900a438db5af13dffdaf8fcd545966d90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  - fiora-predict
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# msms-spectrum-prediction

## Summary

Predict tandem MS/MS spectra from molecular structures using FIORA, a graph neural network that models each bond cleavage via its local molecular neighborhood, and evaluate prediction quality against observed challenge spectra (e.g., CASMI 2016/2022) using intensity coverage and square-root cosine similarity.

## When to use

Use when you need in silico MS/MS spectra for compounds with known SMILES (e.g., CASMI challenge sets, [M+H]+/[M-H]- precursors) and must quantify prediction quality via cosine similarity versus annotated peak-intensity coverage.

## When NOT to use

- Compounds require multi-step (MSn) fragmentation prediction — the article states 'the lack of multi-step fragmentation is currently the biggest limitation' of FIORA
- Predictions need RT or CCS values tuned for production use — RT/CCS prediction is a prototype stage with untuned hyperparameters
- Spectra have missing collision energies or unclear formats — these were explicitly excluded from FIORA's training/evaluation data

## Inputs

- Input CSV with header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' (mgf/msp prediction query format)
- CASMI 2016 challenge spectra with stepped collision energies
- CASMI 2022 challenge spectra (consensus spectra extracted with 10 ppm precursor tolerance and 5 s RT window)
- Observed MS/MS spectra in mgf/msp format for matching and scoring

## Outputs

- Predicted spectra in mgf or msp format
- Per-spectrum annotated peak-intensity coverage values
- Per-spectrum square-root cosine similarity scores (spectral_sqrt_cosine)
- Coverage vs cosine similarity metrics table and joint distribution plots (e.g., casmi_coverage_cosine_metrics.csv, Fig. 6)

## How to apply

Prepare input CSVs with the required header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type' (CASMI 2016 uses stepped collision energies; CASMI 2022 uses consensus spectra extracted with 10 ppm precursor tolerance and a 5 s RT window). Run fiora-predict with --model default to generate predicted spectra in mgf or msp format; for CASMI 16, predict at all 3 collision energies and merge into a single 'stepped' spectrum. Match predicted fragment peaks to observed peaks and compute per-spectrum annotated intensity coverage, enforcing the paper's filters (at least two peak matches, minimum intensity coverage of 50%, maximum precursor intensity of 90%). Score each spectrum with the square-root cosine similarity (spectral_sqrt_cosine) and verify the invariant that cosine similarity is bounded by the square root of intensity coverage; report annotation rates and median coverages (e.g., ~50% annotation / ~80% median coverage for CASMI 16 vs ~25% / ~32% for CASMI 22).

## Related tools

- **FIORA** (Open-source GNN used via the fiora-predict CLI to simulate tandem mass spectra, including fragment ion probabilities and intensities, from SMILES inputs) — https://github.com/BAMeScience/fiora
- **ICEBERG** (State-of-the-art baseline fragmentation predictor (via coleygroup/ms-pred) that FIORA is benchmarked against for prediction quality) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Baseline method evaluated at the closest matching collision energy among 10, 20, 40 eV for comparison with FIORA predictions) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf --model default
```

## Evaluation signals

- Each per-spectrum cosine similarity value must not exceed the square root of its annotated intensity coverage
- Computed annotation rates should fall near the reported values (~50% CASMI 2016, ~25% CASMI 2022)
- Computed median intensity coverages should fall near the reported values (~80% CASMI 2016, ~32% CASMI 2022)
- Spectra scoring checks enforce at least two peak matches, minimum intensity coverage of 50%, and maximum precursor intensity of 90% before evaluation
- For CASMI 2016, spectra predicted at 3 collision energies must be merged into a single 'stepped' spectrum before comparison with observed spectra

## Limitations

- Single-step fragmentation is the biggest limitation; multi-step fragmentation is not implemented
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by lack of experimental metadata, especially collision energies
- Not all collision energies for all CASMI 22 challenge spectra were found during extraction
- Retraining CFM-ID with the authors' dataset was found computationally infeasible, and CFM-ID trained on METLIN17 so train/test separation is not guaranteed for that comparison

## Evidence

- [discussion] Filters for spectral evaluation: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] Stepped CE handling for CASMI 16: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them"
- [discussion] CASMI 22 extraction parameters: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [abstract] Prediction quality benchmark: "FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality"
- [readme] Input CSV format: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
