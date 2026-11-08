---
name: embedding-nearest-neighbor-ranking
description: Use when analyzing preprocessed MSI matrices (e.g. rat brain [X*Y,P]
  data, 198 x 422 pixels) to retrieve nearest-neighbor ions for a query ion image,
  especially for low-SNR isotope discovery where PCC/R2 fail.
when_to_use_negative:
- The MSI data is not preprocessed into a matrix/peak-list pair — the pipeline expects
  [X*Y,P] matrices and [P,1] peak lists, not raw imzML or vendor files.
- The goal is standard quantitative MSI imaging rather than ion colocalization or
  isotope identification — this skill produces ranked neighbor lists, not abundance
  maps.
- Pairwise co-localization statistics alone suffice (high-SNR ions with strong correlation)
  — in that case SIM-based methods like PCC or R2 used in METASPACE and rMSI are adequate,
  and the additional training cost of DeepION is unnecessary.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: DeepION
  role: Main pipeline that trains the SimSiam-based ion image encoder and performs
    the nearest-neighbor similarity ranking in COL or ISO mode
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Contrastive learning framework underlying DeepION that generates self-supervised
    representations without manual annotation
- name: Resnet18
  role: Pretrained backbone network of the twin encoders that extract 512-d ion image
    representation vectors
- name: UMAP
  role: Dimensionality reduction module applied to the 512-d encoder outputs to yield
    20-d embeddings used for distance-based ranking
- name: PyTorch
  role: Implementation library used to train the DeepION model (version 1.8.2 required)
- name: rMSI
  role: Package whose isotope-identification similarity metric (R2) serves as the
    baseline that DeepION is expected to outperform in low-SNR isotope cases
- name: METASPACE
  role: Platform whose isotope-identification similarity metric (PCC) serves as the
    baseline comparison in the ISO-mode evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:62c43f09e96516fbf76e25b0761a8c15af5e40043136f3d04e4afec6ae37f3b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - METASPACE
  - PyTorch
  - Python
  - Resnet18
  - SimSiam
  - UMAP
  - rMSI
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# embedding-nearest-neighbor-ranking

## Summary

Generates dense 20-dimensional DeepION embeddings of ion images from mass spectrometry imaging data via self-supervised contrastive learning (SimSiam/Resnet18), then ranks candidate ions by embedding distance to a query ion for colocalized or isotope identification.

## When to use

Use when analyzing preprocessed MSI matrices (e.g. rat brain [X*Y,P] data, 198 x 422 pixels) to retrieve nearest-neighbor ions for a query ion image, especially for low-SNR isotope discovery where PCC/R2 fail.

## When NOT to use

- The MSI data is not preprocessed into a matrix/peak-list pair — the pipeline expects [X*Y,P] matrices and [P,1] peak lists, not raw imzML or vendor files.
- The goal is standard quantitative MSI imaging rather than ion colocalization or isotope identification — this skill produces ranked neighbor lists, not abundance maps.
- Pairwise co-localization statistics alone suffice (high-SNR ions with strong correlation) — in that case SIM-based methods like PCC or R2 used in METASPACE and rMSI are adequate, and the additional training cost of DeepION is unnecessary.

## Inputs

- Preprocessed MSI matrix data with two-dimensional shape [X*Y,P] (e.g. DATASET/Pos_brain_data_matrix.txt, 198 x 422 pixels)
- Preprocessed MSI peak list data with one-dimensional shape [P,1] (e.g. DATASET/Pos_brain_data_peak.csv)
- MSI data shape (horizontal and vertical pixel counts)
- Mode selection: COL or ISO
- Ion mode: positive or negative

## Outputs

- 20-dimensional ion image representations (min-max scaled to [0,1]) for all ions
- Ranked nearest-neighbor ion lists per query ion (e.g. Pos_COL_result for co-localized ions, Pos_ISO_result for isotope discovery)

## How to apply

Load the preprocessed MSI matrix ([X*Y,P] shape, e.g. 198 x 422 pixels) and peak list ([P,1]) into the DeepION pipeline and train the self-supervised SimSiam model with a pretrained Resnet18 encoder using the modality-specific data augmentation (COL: color jitter, filtering, Poisson noise, random missing value; ISO: additionally intensity-dependent missing value). Pass the learned 512-d representations through the UMAP dimensionality reduction module to obtain 20-dimensional vectors, then apply a min-max scaler to scale features to [0,1]. For each query ion, compute distance-based similarity between its 20-d embedding and all other ion embeddings and retrieve the top-ranked neighbors (--num 5 for COL, full ranking for ISO). The rationale is that dense embeddings capture spectral and spatial ion image structure that intensity-based similarity measures (PCC, R2) miss, particularly for low-SNR isotope partners like m/z 1547.8702 for query 1544.8471 where PCC < 0.5 and R2 < 0.

## Related tools

- **DeepION** (Main pipeline that trains the SimSiam-based ion image encoder and performs the nearest-neighbor similarity ranking in COL or ISO mode) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Contrastive learning framework underlying DeepION that generates self-supervised representations without manual annotation)
- **Resnet18** (Pretrained backbone network of the twin encoders that extract 512-d ion image representation vectors)
- **UMAP** (Dimensionality reduction module applied to the 512-d encoder outputs to yield 20-d embeddings used for distance-based ranking)
- **PyTorch** (Implementation library used to train the DeepION model (version 1.8.2 required))
- **rMSI** (Package whose isotope-identification similarity metric (R2) serves as the baseline that DeepION is expected to outperform in low-SNR isotope cases)
- **METASPACE** (Platform whose isotope-identification similarity metric (PCC) serves as the baseline comparison in the ISO-mode evaluation)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- The 20-d embedding output exists for every ion in the [P,1] peak list and is min-max scaled to [0,1].
- For each COL query ion, the top-ranked neighbors contain the expected co-localized ion species (benchmark accuracy 86.73%, ~25% above the second-best SimSiam variant).
- For ISO queries (e.g. m/z 1544.847 in negative mode), the true isotope partner (e.g. m/z 1547.8702) appears among the top-ranked DeepION neighbors despite PCC < 0.5 and R2 < 0 for that pair.
- The ranking table written to the output file contains a ranked neighbor list for every query ion, using distance-based similarity over the 20-d embeddings.
- Training converges without collapse — the contrastive loss with stop-gradient prevents the encoder from producing degenerate, identical representations for all ions.

## Limitations

- DeepION requires preprocessed matrix input; raw mass-spectrometry imaging files must first be converted to [X*Y,P] matrices with a separate peak list.
- The reduced dimension is fixed to 20 as a trade-off between space-time complexity and information utilization — other dimensionalities are not parameterized.
- The method relies on Poisson-distributed ion abundance and missing-value patterns in MSI; the augmentation strategy (intensity-dependent missing value in ISO mode) may not transfer to data with different acquisition characteristics.
- Training requires a GPU (article used an Nvidia GTX 2080Ti); running on CPU may be impractical for large matrices.
- README specifies strict version pins (python 3.5–3.7, scikit-learn 0.19.1, pytorch 1.8.2, torchvision 0.11.3, kornia 0.4.1, umap 0.5.1) that may conflict with modern environments.

## Evidence

- [readme] DeepION is a deep learning-based low-dimensional representation model of ion images for mass spectrometry imaging.: "DeepION is a deep learning-based low-dimensional representation model of ion images for mass spectrometry imaging."
- [methods] The proposed DeepION model is constructed based on the SimSiam model: "The proposed DeepION model is constructed based on the SimSiam model"
- [methods] Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it: "Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it"
- [methods] The DeepMSI model is implemented in Python using the PyTorch library: "The DeepMSI model is implemented in Python using the PyTorch library"
- [readme] The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data"
- [readme] python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file"
- [abstract] Experimental results of rat brain tissue MSI show that DeepION outperforms other methods for both colocalized ion and isotope ion identification: "Experimental results of rat brain tissue MSI show that DeepION outperforms other methods for both colocalized ion and isotope ion identification"
