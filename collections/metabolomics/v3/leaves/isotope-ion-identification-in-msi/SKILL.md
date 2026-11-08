---
name: isotope-ion-identification-in-msi
description: Use when analyzing preprocessed MSI matrices ([X*Y,P] shape with a [P,1]
  peak list) from rat brain tissue (or similar spatially resolved metabolomics data)
  where you need to find isotope ions of a query monoisotope—especially low-SNR or
  high-missing-value cases (e.g., 88.
when_to_use_negative:
- The analysis goal is identifying colocalized ions from different molecules rather
  than isotope ions from the same molecule—use DeepION's COL mode instead, since the
  ISO mode's intensity-dependent missing-value augmentation is designed specifically
  for isotope ions
- Input data is not a preprocessed MSI matrix in [X*Y,P] format with a corresponding
  [P,1] peak list—DeepION's ISO mode expects this exact preprocessed input structure,
  not raw instrument data
- You need species-level molecular identification from mass spectra alone without
  spatial ion image context—DeepION operates on ion images (spatial abundance maps),
  so it cannot identify isotopes from single spectra without the MSI data
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DeepION
  role: Core ISO-mode isotope identification model with SimSiam-based ResNet18 encoder
    and intensity-dependent missing-value augmentation
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Self-supervised contrastive learning framework underlying DeepION's encoder,
    using a stop-gradient operation to prevent collapse
- name: Resnet18
  role: Pretrained backbone network for DeepION's twin encoders that outputs two 512-dimensional
    representation vectors
- name: UMAP
  role: Dimensionality reduction module applied to encoder outputs, generating a 20-dimensional
    vector O for downstream similarity measurement
- name: PyTorch
  role: Deep learning framework in which the DeepION model is implemented
- name: PCC
  role: SIM-based baseline isotope-identification metric (used in METASPACE) for benchmarking;
    fails on low-SNR cases like m/z 1544.847/1547.8702
- name: R2
  role: SIM-based baseline isotope-identification metric (used in the rMSI package)
    for benchmarking; fails on low-SNR cases
- name: rMSI
  role: R package whose isotope identification approach (R2) is compared as a baseline
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:695ee4b21063dc2e4e82e36addedc8c447cb4d2b4aae73be0b58f192c1d203b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - METASPACE
  - Nvidia GTX 2080Ti
  - PCC
  - PyTorch
  - Python
  - R2
  - Resnet18
  - SimSiam
  - UMAP
  - rMSI
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 36
---

# isotope-ion-identification-in-msi

## Summary

Identify monoisotope–isotope ion pairs in mass spectrometry imaging (MSI) data using DeepION's ISO mode, which trains a SimSiam-based ResNet18 encoder with intensity-dependent missing-value augmentation and measures similarity in a 20-dimensional UMAP-reduced embedding. This discrimination of isotope ions from colocalized ions of different molecules is vital for accurate molecular identification in MSI.

## When to use

Use when analyzing preprocessed MSI matrices ([X*Y,P] shape with a [P,1] peak list) from rat brain tissue (or similar spatially resolved metabolomics data) where you need to find isotope ions of a query monoisotope—especially low-SNR or high-missing-value cases (e.g., 88.62% missing) where PCC and R2 fail.

## When NOT to use

- The analysis goal is identifying colocalized ions from different molecules rather than isotope ions from the same molecule—use DeepION's COL mode instead, since the ISO mode's intensity-dependent missing-value augmentation is designed specifically for isotope ions
- Input data is not a preprocessed MSI matrix in [X*Y,P] format with a corresponding [P,1] peak list—DeepION's ISO mode expects this exact preprocessed input structure, not raw instrument data
- You need species-level molecular identification from mass spectra alone without spatial ion image context—DeepION operates on ion images (spatial abundance maps), so it cannot identify isotopes from single spectra without the MSI data

## Inputs

- Preprocessed MSI data matrix with two-dimensional shape [X*Y,P] where X and Y represent horizontal and vertical pixel numbers and P represents the number of ions (e.g., DATASET/Pos_brain_data_matrix.txt)
- Preprocessed MSI peak data with one-dimensional shape [P,1] (e.g., DATASET/Pos_brain_data_peak.csv)
- Input MSI data shape (e.g., 198 422 for the positive-mode rat brain section)
- Ion mode ('positive' or 'negative') used in the MSI experiments

## Outputs

- Identified monoisotope–isotope pairs per ion mode (e.g., 249 pairs in negative mode, 442 pairs with 92.76% accuracy in positive mode on rat brain MSI)
- Output file listing ISO-mode isotope search results (e.g., Pos_ISO_result)
- Per-query similarity rankings of candidate isotope ions in the 20-dimensional DeepION embedding space
- Comparison table against PCC and R2 baselines showing DeepION's success on low-SNR cases (e.g., m/z 1544.847/1547.8702) where baselines fail

## How to apply

Load the preprocessed MSI matrix (two-dimensional shape [X*Y,P]) and peak list from DATASET/ (e.g., Pos_brain_data_matrix.txt and Pos_brain_data_peak.csv with input_shape 198 422). Configure DeepION in ISO mode by running `python run.py --mode ISO --ion_mode positive` (or negative), which applies T_ISO data augmentation—color jitter, filtering, Poisson noise, random missing value, and an additional intensity-dependent missing value where the missing ratio of ions is proportional to their intensities, designed specifically for isotope ions from the same molecule. Train the SimSiam-based encoder with the pretrained Resnet18 backbone self-supervisedly using the Adam optimizer (learning rate 0.0003, beta1 0.5, beta2 0.99, 100 iterations) with a stop-gradient operation to prevent collapse, then reduce encoder outputs to 20-dimensional vectors via the UMAP module and min–max scale features to [0,1]. For each query ion, compute distance-based similarity ranking of all other ions in the embedding space and retrieve top-ranked neighbors as candidate isotope ions; report the identified monoisotope–isotope pairs to the --output_file (e.g., Pos_ISO_result). Benchmark against PCC and R2 SIM-based baselines (used in METASPACE and the rMSI package) to confirm that DeepION recovers isotope partners like m/z 1547.8702 for query 1544.8471 even when PCC < 0.5 and R2 < 0.

## Related tools

- **DeepION** (Core ISO-mode isotope identification model with SimSiam-based ResNet18 encoder and intensity-dependent missing-value augmentation) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Self-supervised contrastive learning framework underlying DeepION's encoder, using a stop-gradient operation to prevent collapse)
- **Resnet18** (Pretrained backbone network for DeepION's twin encoders that outputs two 512-dimensional representation vectors)
- **UMAP** (Dimensionality reduction module applied to encoder outputs, generating a 20-dimensional vector O for downstream similarity measurement)
- **PyTorch** (Deep learning framework in which the DeepION model is implemented)
- **PCC** (SIM-based baseline isotope-identification metric (used in METASPACE) for benchmarking; fails on low-SNR cases like m/z 1544.847/1547.8702)
- **R2** (SIM-based baseline isotope-identification metric (used in the rMSI package) for benchmarking; fails on low-SNR cases)
- **rMSI** (R package whose isotope identification approach (R2) is compared as a baseline)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- The ISO-mode pair count and accuracy must match the reported values: 249 monoisotope–isotope pairs with 75.90% accuracy in negative mode and 442 pairs with 92.76% accuracy in positive mode on the rat brain MSI datasets
- For the m/z 1544.8471 query, the true isotope partner m/z 1547.8702 must be retrieved among the top-ranked neighbors while PCC < 0.5 and R2 < 0 (baselines fail despite 88.62% and 70.96% missing values respectively)
- The output file (e.g., Pos_ISO_result) must contain ranking columns for DeepION, PCC, and R2 for every query ion in the case studies (m/z 213.902, 214.047, 302.935, 699.493, 718.534, 1544.847/1547.8702)
- The dimensionality reduction module must produce exactly 20-dimensional vectors (a trade-off between space–time complexity and information utilization), min–max scaled to [0,1]
- Training must use the specified Adam optimizer configuration (learning rate 0.0003, beta1 0.5, beta2 0.99, 100 iterations) with the stop-gradient operation active to avoid collapse

## Limitations

- PCC and R2 SIM-based baselines fail to discover the isotope m/z 1547.8702 for m/z 1544.8471 (PCC < 0.5, R2 < 0) despite high proportions of missing values (88.62% and 70.96%), so similarity-metric-only approaches are unreliable at low signal-to-noise
- The reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization; this fixed choice may not be optimal for all MSI datasets
- The model requires a GPU workstation (Nvidia GTX 2080Ti was used); training the SimSiam-based encoder on CPU-only machines may be impractical
- Figure S1 indicates fine differences between colocalized ions and isotope ions that are not included in the main text, making boundary cases between COL and ISO modes harder to judge without consulting the supplementary material

## Evidence

- [intro] Two modes of DeepION, denoted as COL and ISO, are designed for the cases of regular colocalized ions from different molecules and isotope ions from the same molecule, respectively.: "Two modes of DeepION, denoted as COL and ISO, are designed for the cases of regular colocalized ions from different molecules and isotope ions from the same molecule, respectively."
- [readme] while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode.: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
- [results] DeepION in ISO mode obtained 249 monoisotope-isotope pairs with 75.90% accuracy in negative mode and 442 pairs with 92.76% accuracy in positive mode on the rat brain MSI datasets, and detected the isotope m/z 1547.8702 for m/z 1544.8471 where PCC and R2 both failed.: [evidence span withheld — anchor status: not_found]
- [readme] python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file"
- [methods] Train the SimSiam-based encoder with the pretrained Resnet18 backbone in a self-supervised manner using the Adam optimizer (learning rate 0.0003, beta1 0.5, beta2 0.99, 100 iterations) with the stop-gradient operation to prevent collapse.: [evidence span withheld — anchor status: not_found]
