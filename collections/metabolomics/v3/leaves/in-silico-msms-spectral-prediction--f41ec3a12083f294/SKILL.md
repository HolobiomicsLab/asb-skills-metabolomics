---
name: in-silico-msms-spectral-prediction
description: Use when you need simulated MS/MS spectra for compounds lacking reference
  library entries, or when benchmarking fragmentation-prediction quality across NIST/MS-DIAL,
  MSnLib, and CASMI test sets.
when_to_use_negative:
- The query spectra involve multi-stage (MSn) fragmentation beyond MS/MS — single-step
  fragmentation is FIORA's biggest limitation
- You need guaranteed train/test compound separation for CFM-ID comparisons — CFM-ID
  was trained on METLIN17, so this separation is not guaranteed
- Structures exceed the training filter scope (compounds above 1000 Da or collision
  energies above 100 eV were excluded during model training)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FIORA
  role: Graph neural network that predicts MS/MS spectra by modeling bond cleavages
    via local molecular neighborhoods; also estimates RT and CCS
  repo: https://github.com/BAMeScience/fiora
- name: CFM-ID
  role: Competitor fragmentation predictor run via Docker for benchmark comparisons
    at matched collision energies (10, 20, 40 eV)
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ICEBERG
  role: Fragment-level spectral predictor retrained with test compounds excluded,
    used as the second benchmark comparator
  repo: https://github.com/samgoldman97/ms-pred
- name: fiora-eval
  role: CLI for computing spectral_sqrt_cosine similarity between predicted and reference
    spectra, writing per-split result CSVs
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:b189f2f31f8eda75caa0d511ad0b1a5a795e13b977ec1f26021abb7c1eb1d490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - Docker (wishartlab/cfmid)
  - FIORA
  - ICEBERG
  - Python
  - fiora-eval
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# in-silico-msms-spectral-prediction

## Summary

Predict tandem mass spectra (MS/MS) from molecular structures using graph-neural-network fragmentation simulators (FIORA, with ICEBERG and CFM-ID as comparators), enabling spectral reference library expansion and benchmarking of prediction quality.

## When to use

Use when you need simulated MS/MS spectra for compounds lacking reference library entries, or when benchmarking fragmentation-prediction quality across NIST/MS-DIAL, MSnLib, and CASMI test sets.

## When NOT to use

- The query spectra involve multi-stage (MSn) fragmentation beyond MS/MS — single-step fragmentation is FIORA's biggest limitation
- You need guaranteed train/test compound separation for CFM-ID comparisons — CFM-ID was trained on METLIN17, so this separation is not guaranteed
- Structures exceed the training filter scope (compounds above 1000 Da or collision energies above 100 eV were excluded during model training)

## Inputs

- Query CSV with columns Name, SMILES, Precursor_type, CE, Instrument_type
- Reference MS/MS spectra (NIST'17 + MS-DIAL .msp libraries, MSnLib mgf, CASMI 2016/2022 challenge spectra)
- Default FIORA model checkpoint from fiora/resources/models
- CFM-ID v4 pre-trained model via Docker (wishartlab/cfmid)
- Retrained ICEBERG checkpoint from the ms-pred repository

## Outputs

- Predicted spectra in msp or mgf format (e.g., example_spec.mgf)
- Per-split evaluation CSVs of cosine similarities (e.g., test_eval.csv)
- Aggregated benchmark table of median cosine similarities per model, test set, and ionization mode (table1_spectral_similarity.csv)

## How to apply

Assemble a query CSV with the header columns 'Name', 'SMILES', 'Precursor_type', 'CE', 'Instrument_type', filtered to ionization types [M+H]+ and [M-H]- (plus [M]+/[M]- for MSnLib). Run fiora-predict with the default packaged model from fiora/resources/models (or --model default), optionally enabling RT/CCS prediction and a GPU via --dev cuda:0 for ~30x speedup (~10,000 spectra in under five minutes). For comparison, generate CFM-ID v4 predictions via the wishartlab/cfmid Docker container and pick the prediction at the closest matching collision energy among 10, 20, and 40 eV; run retrained ICEBERG from the ms-pred repository with test compounds excluded from its training data. Handle benchmark specifics: for CASMI 2016, merge FIORA predictions across the three stepped collision energies into a single 'stepped' spectrum; for CASMI 2022, extract challenge spectra with OpenMS using 10 ppm precursor tolerance and a 5-second retention time window, merging replicate measurements at the same collision energy into consensus spectra. Evaluate with fiora-eval using the spectral_sqrt_cosine metric (square-root-transformed intensities including the precursor peak) and write per-split result CSVs to --output-dir.

## Related tools

- **FIORA** (Graph neural network that predicts MS/MS spectra by modeling bond cleavages via local molecular neighborhoods; also estimates RT and CCS) — https://github.com/BAMeScience/fiora
- **CFM-ID** (Competitor fragmentation predictor run via Docker for benchmark comparisons at matched collision energies (10, 20, 40 eV)) — https://hub.docker.com/r/wishartlab/cfmid
- **ICEBERG** (Fragment-level spectral predictor retrained with test compounds excluded, used as the second benchmark comparator) — https://github.com/samgoldman97/ms-pred
- **fiora-eval** (CLI for computing spectral_sqrt_cosine similarity between predicted and reference spectra, writing per-split result CSVs) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i examples/example_input.csv -o examples/example_spec.mgf --dev cuda:0
```

## Evaluation signals

- Predicted spectra output validates against the expected results file (examples/expected_output.mgf); running pytest -v tests performs this verification automatically
- Median spectral_sqrt_cosine similarity of FIORA predictions exceeds ICEBERG and CFM-ID on the test split, MSnLib, and CASMI 16, outperforming the runner-up by 10% to 49%
- FIORA GPU runtime is on the order of ~7 minutes for 14,100 spectra (~0.03 s per prediction); every tool/device combination has a complete total-time and per-spectrum entry
- Each evaluation row has at least two peak matches between predicted and reference spectrum, mirroring the training filter criteria
- All test-set compounds were removed from the training data for FIORA and ICEBERG to preserve test integrity

## Limitations

- Single-step fragmentation is FIORA's biggest limitation; multi-step fragmentation is not implemented
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by missing experimental metadata, especially collision energies
- Not all collision energies for all CASMI 2022 challenge spectra were found during extraction
- Retraining CFM-ID with the authors' dataset was computationally infeasible, and CFM-ID's METLIN17 training means train/test compound separation is not guaranteed
- MSnLib support assumes .msp/.mgf inputs; the packaged downloader defaults to MSnLib v7 on Zenodo record 16984129

## Evidence

- [abstract] FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality: "FIORA not only surpasses state-of-the-art fragmentation algorithms, ICEBERG and CFM-ID, in prediction quality"
- [discussion] In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single "stepped" spectrum.: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single “stepped" spectrum."
- [discussion] Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds.: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [readme] Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type": "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
- [readme] By default, an open-source model is selected automatically, and predictions typically complete within a few seconds. For faster performance, specify a GPU device using the --dev option: "By default, an open-source model is selected automatically, and predictions typically complete within a few seconds. For faster performance, specify a GPU device using the `--dev` option"
