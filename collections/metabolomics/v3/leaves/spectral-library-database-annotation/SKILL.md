---
name: spectral-library-database-annotation
description: Use when MS2 (MSn) tandem mass spectra must be annotated with candidate
  molecular structures from spectral databases, and prediction quality must be quantified
  at exact-match, Tanimoto >0.9, and Tanimoto >0.8 thresholds.
when_to_use_negative:
- Spectra were recorded in negative ion mode — TeFT was trained only on positive ion
  mode (H+) spectra, restricting its applicability
- High-resolution, high-mass-accuracy spectra where fingerprint-prediction or large-scale
  models (SIRIUS4/CSI:FingerID, MSNovelist, MassGenie) may outperform TeFT's low-resolution
  design
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TeFT (Transformer enabled Fragment Tree)
  role: Generates candidate molecular structures de novo from MS2 spectra by combining
    fragmentation tree models with a deep learning Transformer that predicts SMILES
    fragment strings
  repo: https://github.com/thumingo/TeFT
- name: SIRIUS4
  role: Baseline method whose exact-match (27.6%) and >0.9 Tanimoto (39%) results
    benchmark TeFT's annotation performance
- name: Transformer module
  role: Component that generates the complete molecular structure (SMILES fragment
    strings) from the spectral representation produced by the fragmentation tree models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:f313b3bf0456654ce446f36a0e32c87b2f0c82114eab73e63f08bd3bf8f10421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - SIRIUS
  - SIRIUS4
  - TeFT (Transformer enabled Fragment Tree)
  - Transformer
  - Transformer module
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spectral-library-database-annotation

## Summary

Evaluates structure annotation of MS2 spectra by running the TeFT (Transformer enabled Fragment Tree) pipeline over spectra compiled from open-source libraries (GNPS, HMDB 5.0, MoNA) and scoring predicted versus actual structures with MACCS-key and Morgan fingerprint Tanimoto similarity.

## When to use

Use when MS2 (MSn) tandem mass spectra must be annotated with candidate molecular structures from spectral databases, and prediction quality must be quantified at exact-match, Tanimoto >0.9, and Tanimoto >0.8 thresholds.

## When NOT to use

- Spectra were recorded in negative ion mode — TeFT was trained only on positive ion mode (H+) spectra, restricting its applicability
- High-resolution, high-mass-accuracy spectra where fingerprint-prediction or large-scale models (SIRIUS4/CSI:FingerID, MSNovelist, MassGenie) may outperform TeFT's low-resolution design

## Inputs

- 660 non-repetitive MS2 spectra drawn from GNPS, HMDB (5.0), and MoNA spectral libraries
- Ground-truth molecular structures (as SMILES) corresponding to each test spectrum
- TeFT model checkpoint from the thumingo/TeFT repository

## Outputs

- Per-spectrum best candidate SMILES structure predictions
- MACCS-key and Morgan fingerprint Tanimoto similarity scores (predicted vs. real structure)
- results_table.csv with exact-match, >0.9, and >0.8 similarity counts and percentages (expected: 195, 311, 439 of 660)

## How to apply

Compile non-repetitive MS2 test spectra from the open-source spectrometry databases GNPS, HMDB (5.0), and MoNA — in this benchmark 660 randomly extracted spectra. Run the TeFT pipeline (thumingo/TeFT), which combines fragmentation tree models with a deep learning Transformer module that generates SMILES de novo, to obtain the best candidate structure per spectrum. Convert predicted and real structures to SMILES and compute Tanimoto similarity over MACCS-key and Morgan fingerprints. Count exact matches (Tanimoto = 1) and candidates above the 0.9 and 0.8 thresholds, then report percentages against the reference outcome: 195/660 (30%) exact, 311/660 (47%) >0.9, 439/660 (67%) >0.8, with SIRIUS4 comparison at 27.6% exact and 39% >0.9. Write aggregate metrics to results_table.csv.

## Related tools

- **TeFT (Transformer enabled Fragment Tree)** (Generates candidate molecular structures de novo from MS2 spectra by combining fragmentation tree models with a deep learning Transformer that predicts SMILES fragment strings) — https://github.com/thumingo/TeFT
- **SIRIUS4** (Baseline method whose exact-match (27.6%) and >0.9 Tanimoto (39%) results benchmark TeFT's annotation performance)
- **Transformer module** (Component that generates the complete molecular structure (SMILES fragment strings) from the spectral representation produced by the fragmentation tree models)

## Evaluation signals

- Exact-match count equals 195/660 (30% of spectra) with predicted Tanimoto similarity of 1 against the real structure
- Counts of candidates exceeding Tanimoto 0.9 and 0.8 equal 311 (47%) and 439 (67%), respectively, out of 660
- Tanimoto similarities are computed from both MACCS-key and Morgan fingerprints after converting predicted and real structures to SMILES
- Results table reproduces the reported SIRIUS4 baseline figures of 27.6% exact matches and 39% above 0.9 similarity
- Each test spectrum traces back to one of the three open-source sources (GNPS, HMDB 5.0, MoNA) with no duplicate spectra in the 660-spectrum set

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting applicability to negative-mode or other adduct data
- On high-quality/high-resolution spectra, the model may not show performance advantages over other deep models such as SIRIUS4 (CSI:FingerID), MSNovelist, or MassGenie
- Applied constraints narrow the hypothesis space: molecular weight <500 Da, only ten permitted elements (C, H, O, N, P, S, Cl, Br, I, F), 1500-node SMILES tree limit, and top 5–6 fragmentation peaks as experimental data

## Evidence

- [abstract] we propose the Transformer enabled Fragment Tree (TeFT) method, which combines various types of fragmentation tree models and a deep learning Transformer module: "we propose the Transformer enabled Fragment Tree (TeFT) method, which combines various types of fragmentation tree models and a deep learning Transformer module"
- [discussion] The training set utilized for the Transformer model was created by compiling open-source spectrometry databases GNPS, HMDB (5.0), and MoNA: "compiling open-source spectrometry databases GNPS, HMDB (5.0), and MoNA"
- [discussion] We propose a novel prediction model, TeFT, designed for de novo structure generation from low-resolution tandem mass spectrometry (MSn) spectra: "We propose a novel prediction model, TeFT, designed for de novo structure generation from low-resolution tandem mass spectrometry (MSn) spectra"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
