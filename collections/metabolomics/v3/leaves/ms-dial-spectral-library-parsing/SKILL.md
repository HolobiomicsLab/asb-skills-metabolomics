---
name: ms-dial-spectral-library-parsing
description: Use when a public .msp-format spectral library (MS-DIAL) must be mined
  for compounds with RT or CCS annotations, e.g. to build training sets for retention-time
  or CCS predictors that NIST'17 cannot supply.
when_to_use_negative:
- 'The target library is NIST''17: it does not provide RT and CCS values of the measured
  compounds, so it cannot supply the RT/CCS annotations this skill extracts.'
- The task is pure MS/MS spectral intensity prediction from MSnLib, which has its
  own download/preprocess scripts (download_msnlib.py, preprocess_msnlib.py) and does
  not need MS-DIAL RT/CCS mining.
tools:
- name: FIORA
  role: 'Downstream consumer: trains RT/CCS submodules on the parsed MS-DIAL subsets
    (fiora-train) and generates RT/CCS predictions (fiora-predict --rt --ccs)'
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:7edbd90f0cf3c81f61fcc24131086ed76a9a0e30a51c587d5f0da5b745b54b0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ms-dial-spectral-library-parsing

## Summary

Parse the public MS-DIAL spectral library (.msp format) to extract compounds annotated with retention time (RT) and collision cross section (CCS) values, forming the RT and CCS training/evaluation subsets used to train and benchmark FIORA's RT and CCS prediction submodules.

## When to use

Use when a public .msp-format spectral library (MS-DIAL) must be mined for compounds with RT or CCS annotations, e.g. to build training sets for retention-time or CCS predictors that NIST'17 cannot supply.

## When NOT to use

- The target library is NIST'17: it does not provide RT and CCS values of the measured compounds, so it cannot supply the RT/CCS annotations this skill extracts.
- The task is pure MS/MS spectral intensity prediction from MSnLib, which has its own download/preprocess scripts (download_msnlib.py, preprocess_msnlib.py) and does not need MS-DIAL RT/CCS mining.

## Inputs

- MS-DIAL spectral library in .msp format (public, annotated MS/MS spectra with RT and CCS values)
- Compound metadata: SMILES, MOL format, InChiKeys, precursor type, collision energy
- FIORA input CSV with columns Name, SMILES, Precursor_type, CE, Instrument_type

## Outputs

- 409-compound RT-annotated subset and 1346-compound CCS-annotated subset (80% train / 10% validation / 10% test)
- Trained FIORA RT and CCS submodule checkpoints (e.g. checkpoints/fiora.pt)
- RT and CCS predictions for validation/test compounds and CASMI 2016 compounds, with R2 parity plots (Fig. 5)

## How to apply

Read the MS-DIAL .msp library (a public annotated MS/MS collection, unlike NIST'17 which provides no RT/CCS values) and extract compounds carrying RT annotations (409-compound subset) and CCS annotations (1346-compound subset). Apply the paper's data-cleaning filters: drop compounds with inconsistencies between SMILES, MOL format and InChiKeys, filter ionization types to [M+H]+ and [M-H]-, enforce a maximum weight of 1000 Da and maximum collision energy of 100 eV, and require at least two peak matches, minimum peak intensity coverage of 50%, and maximum precursor intensity of 90% (soft filters: 50% of total peaks or at least five peaks matched, or intensity coverage of 0.8). Split each subset 80% for training with the remaining 20% split evenly between validation and testing, then train the FIORA RT/CCS submodules via fiora-train (200 epochs, ADAM, weighted MSE loss, plateau-reducing learning-rate scheduler) and predict with fiora-predict --rt --ccs. Judge success by R2 parity against baselines: RT R2 of about 0.70, CCS R2 of about 0.93 on the test split and about 0.86 on CASMI 16 compounds versus a precursor-m/z linear regression baseline (0.90/0.80).

## Related tools

- **FIORA** (Downstream consumer: trains RT/CCS submodules on the parsed MS-DIAL subsets (fiora-train) and generates RT/CCS predictions (fiora-predict --rt --ccs)) — https://github.com/BAMeScience/fiora

## Examples

```
fiora-predict -i queries.csv -o predictions.mgf --rt --ccs
```

## Evaluation signals

- Subset sizes match expectations: 409 compounds with RT annotations and 1346 compounds with CCS annotations extracted from the .msp library.
- Split composition matches the protocol: 80% training, remaining 20% split evenly between validation and testing.
- Filtered set shows no compounds with inconsistencies between SMILES, MOL format and InChiKeys.
- CCS prediction reaches R2 of about 0.93 on the test split and about 0.86 on CASMI 16 compounds, beating the precursor-m/z linear regression baseline (0.90/0.80).
- RT predictions reach R2 of about 0.70 on the test split versus the logP-based linear regression baseline.

## Limitations

- Hyperparameters were not specifically tuned for the RT and CCS prediction task; the feature is still in prototype stage.
- FIORA's RT/CCS training set is relatively small (about 10,000 unique compounds overall), limited by lack of experimental metadata, especially collision energies.
- MS/MS spectra with missing collision energies or unclear formats were excluded from the parsed library.

## Evidence

- [discussion] MS-DIAL library source: "we process a public library available in .msp format on the MS-DIAL website36, which contains a collection of annotated MS/MS spectra"
- [discussion] NIST lacks RT/CCS: "The NIST’17 MS/MS library does not provide information about RT and CCS values of the measured compounds."
- [discussion] compound consistency filter: "Compounds with inconsistencies between SMILES, MOL format and InChiKeys were filtered out immediately."
- [discussion] split protocol: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [readme] FIORA RT/CCS capability: "Using a graph neural network, FIORA models bond cleavages, fragment intensities, and estimates retention times (RT) and collision cross sections (CCS)."
- [readme] fiora-predict CLI supports RT/CCS flags: "fiora-predict [-h] -i INPUT -o OUTPUT [--model MODEL] [--rt | --no-rt] [--ccs | --no-ccs] [--annotation | --no-annotation]"
