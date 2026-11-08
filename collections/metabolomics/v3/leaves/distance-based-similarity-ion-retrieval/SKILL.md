---
name: distance-based-similarity-ion-retrieval
description: Use when you have a preprocessed MSI data matrix [X*Y,P] plus peak list
  [P,1] and need to retrieve colocalized or isotope ion partners by similarity of
  learned ion-image representations rather than raw spectral correlation.
when_to_use_negative:
- Input is raw unprocessed instrument data rather than a preprocessed [X*Y,P] MSI
  matrix with a [P,1] peak list
- The spatial shape of the ion images (X,Y) is unknown, since the 2D ion-image structure
  is required for representation learning
- The goal is mass-based isotope annotation from exact m/z deltas alone rather than
  similarity of spatial ion-image distributions
tools:
- name: DeepION
  role: Self-supervised contrastive model that generates the low-dimensional ion image
    representations used for distance-based similarity measurement
  repo: https://github.com/BioNet-XMU/DeepION
- name: UMAP
  role: Dimensionality-reduction module that reduces 512-D encoder outputs to 20-D
    vectors before similarity measurement
- name: SimSiam
  role: Architecture on which DeepION is built; twin ResNet18 encoders with stop-gradient
    training produce the representation vectors
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:953907f594fe9cad6cdd4ac4d81037c2df31e04bdd7bccfa01db5f01e1c415fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - METASPACE
  - Nvidia GTX 2080Ti
  - PCC
  - Python
  - R2
  - Resnet18
  - SimSiam
  - UMAP
  - rMSI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# distance-based-similarity-ion-retrieval

## Summary

A retrieval procedure that ranks ion images by distance in the low-dimensional (20-D) representation space learned by DeepION, enabling identification of colocalized ions (COL mode) and monoisotope-isotope pairs (ISO mode) in mass spectrometry imaging data.

## When to use

Use when you have a preprocessed MSI data matrix [X*Y,P] plus peak list [P,1] and need to retrieve colocalized or isotope ion partners by similarity of learned ion-image representations rather than raw spectral correlation.

## When NOT to use

- Input is raw unprocessed instrument data rather than a preprocessed [X*Y,P] MSI matrix with a [P,1] peak list
- The spatial shape of the ion images (X,Y) is unknown, since the 2D ion-image structure is required for representation learning
- The goal is mass-based isotope annotation from exact m/z deltas alone rather than similarity of spatial ion-image distributions

## Inputs

- Preprocessed MSI data matrix with 2D shape [X*Y,P] (e.g. DATASET/Pos_brain_data_matrix.txt)
- Preprocessed MSI peak list with shape [P,1] (e.g. DATASET/Pos_brain_data_peak.csv)
- Input MSI data shape (e.g. 198 422 for rat brain positive mode)
- Mode selection: COL or ISO
- Ion mode: positive or negative
- Number of searched colocalized ions per ion (COL mode)

## Outputs

- Per-ion lists of retrieved colocalized ions (COL mode) or monoisotope-isotope pairs (ISO mode)
- Named output file (e.g. Pos_ISO_result or Pos_COL_result)
- 20-D min-max scaled ion image representation vectors used for distance measurement

## How to apply

First represent each ion image with DeepION: self-supervised SimSiam training with a pretrained ResNet18 backbone (Adam, lr 0.0003, beta1 0.5, beta2 0.99, 100 iterations, stop-gradient to prevent collapse). Pass the 512-D encoder outputs through a UMAP dimensionality-reduction module down to 20-D vectors, chosen as a trade-off between space-time complexity and information utilization, then apply a min-max scaler to adjust features to [0,1]. Run similarity measurement in this reduced space in the chosen mode: COL for regular colocalized ions from different molecules (with a --num parameter for number of searched ions per ion) or ISO for isotope ions from the same molecule, where the intensity-dependent missing-value augmentation treats missing ratio as proportional to ion intensity. Select --ion_mode (positive/negative) consistent with the experiment, e.g. input_shape 198 422 for the rat brain positive-mode section, and write results to the named output file.

## Related tools

- **DeepION** (Self-supervised contrastive model that generates the low-dimensional ion image representations used for distance-based similarity measurement) — https://github.com/BioNet-XMU/DeepION
- **UMAP** (Dimensionality-reduction module that reduces 512-D encoder outputs to 20-D vectors before similarity measurement)
- **SimSiam** (Architecture on which DeepION is built; twin ResNet18 encoders with stop-gradient training produce the representation vectors) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- ISO-mode retrieval on the rat brain datasets should reproduce 249 monoisotope-isotope pairs in negative mode and 442 pairs with 92.76% accuracy in positive mode
- Retrieved representation vectors must be 20-dimensional and min-max scaled to the [0,1] range
- The method should recover m/z 1547.8702 as the isotope of m/z 1544.8471, a pair that PCC and R2 baselines both failed to discover
- COL-mode classification accuracy on the benchmark colocalization task should reach about 86.73%, roughly 25% above the second-best SimSiam baseline
- Output file must contain pair/retrieval results consistent with the requested --mode, --ion_mode, and --num parameters

## Limitations

- Performance depends on the modality-specific augmentation design; ISO mode relies on intensity-dependent missing values being appropriate for isotope ions from the same molecule
- Reduced dimension fixed at 20 is a trade-off between space-time complexity and information utilization, not an optimal choice for every dataset
- Requires a GPU workstation (Nvidia GTX 2080Ti reported) and specific package versions (pytorch 1.8.2, umap 0.5.1, scikit-learn 0.19.1, python 3.5-3.7)

## Evidence

- [abstract] which further facilitates the distance-based similarity measurement for the identification of colocalized ions: "which further facilitates the distance-based similarity measurement for the identification of colocalized ions"
- [methods] Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it: "Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it"
- [methods] a min-max scaler is adopted to adjust the range of features to [0, 1]: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [readme] Two modes of DeepION, denoted as COL and ISO are designed for the cases of regular co-localized ions from different molecules and isotope ions from a same molecule respectively.: "Two modes of DeepION, denoted as “COL” and “ISO” are designed for the cases of regular co-localized ions from different molecules and isotope ions from a same molecule respectively."
- [methods] the reduced dimension is set to 20 by a trade-off between space-time complexity and information utilization: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
