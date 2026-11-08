---
name: train-test-structural-similarity-binning
description: Use when benchmarking MS/MS spectral predictors (e.g., FIORA, ICEBERG,
  CFM-ID) and you need to test whether prediction quality, measured by sqrt-weighted
  cosine similarity, degrades for test compounds structurally dissimilar from the
  training set.
when_to_use_negative:
- 'Comparisons where CFM-ID''s train/test compound separation matters: CFM-ID was
  trained on METLIN17, so separation from the FIORA/MSnLib test split is not guaranteed
  and low-Tanimoto bins may be contaminated'
- Analysis of multi-stage MSn or spectra lacking collision energies, since FIORA only
  supports single-step fragmentation and spectra with missing collision energies were
  excluded
- Datasets without parseable SMILES structures, since Morgan fingerprints and Tanimoto
  similarity cannot be computed
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: FIORA
  role: Predicts MS/MS spectra for test compounds and provides fiora-eval with the
    spectral_sqrt_cosine metric used as the per-compound score
  repo: https://github.com/BAMeScience/fiora
- name: ICEBERG
  role: Baseline fragmentation model whose binned cosine similarities are compared
    against FIORA across Tanimoto intervals
  repo: https://github.com/samgoldman97/ms-pred
- name: CFM-ID
  role: Baseline fragmentation model run under matching collision energies whose binned
    cosine similarities are compared against FIORA
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: RDKit
  role: Computes Morgan fingerprints (2048 bits, radius 3) and maximum Tanimoto similarity
    of each test compound against all training compounds
- name: MSnLib
  role: Source of the preprocessed spectral library (SMILES, precursor types, collision
    energies) supplying the test and training splits
  repo: https://zenodo.org/records/16984129
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:92e9c69a5cbe71246415718142cd2d3f576a69069905585de51b9d3bbfd9a3de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  - MSnLib
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# train-test-structural-similarity-binning

## Summary

Quantifies how MS/MS spectral prediction quality depends on structural novelty by binning test compounds by their maximum Tanimoto similarity to the training set and computing per-bin median spectral similarity (with IQR). Used in FIORA (Fig. 3) to demonstrate generalizability to compounds structurally dissimilar from the training set.

## When to use

Use when benchmarking MS/MS spectral predictors (e.g., FIORA, ICEBERG, CFM-ID) and you need to test whether prediction quality, measured by sqrt-weighted cosine similarity, degrades for test compounds structurally dissimilar from the training set.

## When NOT to use

- Comparisons where CFM-ID's train/test compound separation matters: CFM-ID was trained on METLIN17, so separation from the FIORA/MSnLib test split is not guaranteed and low-Tanimoto bins may be contaminated
- Analysis of multi-stage MSn or spectra lacking collision energies, since FIORA only supports single-step fragmentation and spectra with missing collision energies were excluded
- Datasets without parseable SMILES structures, since Morgan fingerprints and Tanimoto similarity cannot be computed

## Inputs

- Preprocessed MSnLib spectral library (resources/data/msnlib/library.csv) with SMILES, precursor types, and collision energies for [M+H]+ test and training splits
- Trained FIORA model checkpoint (checkpoints/fiora.pt)
- Predicted spectra from FIORA, ICEBERG, and CFM-ID under matching collision energies and instrument type (e.g., HCD)
- Reference MS/MS spectra from the MSnLib test split

## Outputs

- Per-bin median cosine similarity and interquartile range per tool (tanimoto_bin_summary.csv)
- Binned median cosine similarity plot with IQR bands (fig3_tanimoto_cosine_similarity.png)

## How to apply

Load the preprocessed MSnLib spectral library (resources/data/msnlib/library.csv) containing SMILES, precursor types, and collision energies for the positive-mode [M+H]+ test and training splits. Compute Morgan fingerprints (2048 bits, radius 3) with RDKit and, for each test compound, its maximum Tanimoto similarity against all training compounds to quantify structural novelty. Predict MS/MS spectra for every positive-mode test compound with FIORA (fiora-predict, checkpoints/fiora.pt), ICEBERG, and CFM-ID under matching collision energies and instrument type. Score each prediction against its reference spectrum using the sqrt-weighted cosine similarity (spectral_sqrt_cosine, FIORA's default metric via fiora-eval). Bin compounds into fixed Tanimoto similarity intervals and compute the per-bin median cosine similarity and interquartile range per tool, rendering the binned plot with IQR bands (fig3_tanimoto_cosine_similarity.png, tanimoto_bin_summary.csv). The rationale is that low-Tanimoto bins isolate generalization: FIORA remains robust even for the most dissimilar compounds.

## Related tools

- **FIORA** (Predicts MS/MS spectra for test compounds and provides fiora-eval with the spectral_sqrt_cosine metric used as the per-compound score) — https://github.com/BAMeScience/fiora
- **ICEBERG** (Baseline fragmentation model whose binned cosine similarities are compared against FIORA across Tanimoto intervals) — https://github.com/samgoldman97/ms-pred
- **CFM-ID** (Baseline fragmentation model run under matching collision energies whose binned cosine similarities are compared against FIORA) — https://hub.docker.com/r/wishartlab/cfmid
- **RDKit** (Computes Morgan fingerprints (2048 bits, radius 3) and maximum Tanimoto similarity of each test compound against all training compounds)
- **MSnLib** (Source of the preprocessed spectral library (SMILES, precursor types, collision energies) supplying the test and training splits) — https://zenodo.org/records/16984129

## Examples

```
from rdkit import Chem; from rdkit.Chem import AllChem; fp = AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smiles), 3, nBits=2048)
```

## Evaluation signals

- FIORA's median cosine similarity in the 0.2-0.3 Tanimoto bin must reproduce the reported value of 0.63
- FIORA's median cosine stays at or above 0.8 for Tanimoto bins in the 0.6-1 range
- FIORA's binned median cosine is above ICEBERG and CFM-ID across all Tanimoto bins
- Each Tanimoto bin's summary statistics (median and IQR) must be computed over the same test compounds scored by all three tools under matching collision energies
- Summary CSV row count and bin edges must cover the full observed Tanimoto range with no test compounds dropped unannotated

## Limitations

- CFM-ID was trained on the METLIN17 library, so train/test compound separation is not guaranteed for CFM-ID comparisons in low-similarity bins
- FIORA was trained on a relatively small dataset of ~10,000 unique compounds, limited by lack of experimental metadata, especially collision energies
- FIORA's single-step fragmentation (lack of multi-step fragmentation) is its biggest limitation and may affect predictions for compounds requiring secondary fragmentations

## Evidence

- [intro] Objective: benchmark FIORA against top-performing methods CFM-ID and ICEBERG and demonstrate generalizability to compounds structurally dissimilar from the training set.: [evidence span withheld — anchor status: not_found]
- [discussion] Train/test split definition for the spectral library underlying the binning analysis.: "FIORA was trained on 80% of the molecular structures, with the remaining 20% split evenly between validation and testing."
- [discussion] Caveat affecting CFM-ID's position in low-Tanimoto bins.: "CFM-ID was trained on the METLIN17 library, which means that this separation is not guaranteed."
- [readme] MSnLib library and preprocessing scripts provide the split annotations used for fingerprinting.: "Scripts for downloading and preprocessing MSnLib are provided in `resources/data/msnlib` (`download_msnlib.py` and `preprocess_msnlib.py`)."
