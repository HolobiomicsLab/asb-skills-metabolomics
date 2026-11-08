---
name: ms2-product-ion-mass-difference-alignment
description: Use when comparing a query MS/MS spectrum against reference library spectra
  to find structural analogues, and you need to explain which query–reference product-ion
  pairs (including mass-shifted ones like +15.99 Da) drive the similarity score.
when_to_use_negative:
- You only need a simple heuristic alignment score without interpretability — plain
  Cosine or Modified Cosine suffices and needs no trained model
- No trained TransExION model or structural-similarity reference files are available
  and training data (GNPS_MassBank_train/val MGF) cannot be obtained
- The spectra are not tandem MS/MS product-ion spectra convertible to the MGF/h5py
  format the TransExION scripts expect
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TransExION
  role: Transformer-based model that predicts MS/MS spectral similarity considering
    mass differences between query and reference product ions, and provides the post-hoc
    relevance explanation
  repo: https://github.com/banhdzui/TransExION
- name: PyTorch
  role: Framework in which the TransExION model and the explanation scripts are implemented
    and executed
- name: Matplotlib
  role: Rendering the product-ion relevance matrices as heatmap images
- name: h5py
  role: Storage format for spectra and spectrum-pair id files consumed and produced
    by the conversion and explanation scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:f3455e320d525a55a2866916c0518e3e12b8e2d9d5f47ea9a08a4bc1c334fb68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - Matplotlib
  - PyTorch
  - TransExION
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ms2-product-ion-mass-difference-alignment

## Summary

Aligns MS/MS product ions between a query spectrum and a reference spectrum not only by co-occurring fragments and neutral losses but also through their mass difference, using a Transformer-based model (TransExION) with a post-hoc layer-wise relevance propagation explanation rendered as a query-vs-reference product-ion heatmap.

## When to use

Use when comparing a query MS/MS spectrum against reference library spectra to find structural analogues, and you need to explain which query–reference product-ion pairs (including mass-shifted ones like +15.99 Da) drive the similarity score.

## When NOT to use

- You only need a simple heuristic alignment score without interpretability — plain Cosine or Modified Cosine suffices and needs no trained model
- No trained TransExION model or structural-similarity reference files are available and training data (GNPS_MassBank_train/val MGF) cannot be obtained
- The spectra are not tandem MS/MS product-ion spectra convertible to the MGF/h5py format the TransExION scripts expect

## Inputs

- Query MS/MS spectra file in h5py format (e.g. GNPS_MassBank_test.db, converted from MGF via ms_convert_mgf2h5py.py)
- Reference/library MS/MS spectra file in h5py format (e.g. GNPS_MassBank_train.db)
- Spectrum pair id file in h5py format (converted from CSV via ms_covert_csv2h5py.py) specifying query–reference pairs to explain
- Pre-trained TransExION model file (e.g. GNPS_MassBank.ms.model from Zenodo record 8175528)

## Outputs

- Heatmap image per spectrum pair showing the relevance matrix of query vs reference product ions with their mass differences
- Predicted spectral similarity scores (CSV, e.g. GNPS_MassBank_result.csv when using ms_test_spectral_similarity.py)

## How to apply

Convert MGF spectra to h5py format with ms_convert_mgf2h5py.py, then run the trained TransExION model (e.g. GNPS_MassBank.ms.model, trained on GNPS/MassBank data) via ms_explain_spectral_similarity.py on a spectrum-pairs h5py id file built with ms_covert_csv2h5py.py. The model's key novelty is that, besides aligned product ions and neutral losses, the mass difference between query and reference fragments is used for similarity scoring, letting it detect related fragments that differ by a modification such as one oxygen (+15.99 Da). Layer-wise relevance propagation computes pairwise relevance between query and reference product ions; per pair, retain the top q=20 query product ions by overall contribution and the top k=3 most associated reference product ions per query ion. Render the resulting relevance matrix as a heatmap whose cells show the query ion, reference ion(s) and their mass difference, and check that expected offsets (e.g. m/z 163.06→179.03 at +15.99 Da for DL-beta-Homophenylalanine; 157.13→171.15 at -14.02 Da for Scopularide F) are recovered.

## Related tools

- **TransExION** (Transformer-based model that predicts MS/MS spectral similarity considering mass differences between query and reference product ions, and provides the post-hoc relevance explanation) — https://github.com/banhdzui/TransExION
- **PyTorch** (Framework in which the TransExION model and the explanation scripts are implemented and executed)
- **Matplotlib** (Rendering the product-ion relevance matrices as heatmap images)
- **h5py** (Storage format for spectra and spectrum-pair id files consumed and produced by the conversion and explanation scripts)

## Examples

```
python ms_explain_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --pairs pairs.db --model GNPS_MassBank.ms.model --output heatmaps/
```

## Evaluation signals

- Each generated heatmap file for a queried pair exists and is a non-empty image
- The DL-beta-Homophenylalanine pair's heatmap reproduces the +15.99 Da offset associations (m/z 163.06→179.03, 145.06→161.05, 120.06→136.04) as in Fig. 8 of the paper
- The Scopularide F (cyclic lipodepsipeptide) pair's heatmap reproduces the -14.02 Da offset associations (m/z 157.13→171.15, 351.26→365.29) as in Fig. 9
- Relevance matrices are filtered to the top q=20 query product ions by overall contribution, with at most k=3 reference product ions retained per query ion
- Predicted spectral similarities from the model outperform Cosine, Modified Cosine, Spec2Vec and MS2DeepScore baselines on the GNPS/MassBank test sets, especially at high Tanimoto distance cut-offs

## Limitations

- The explanation is post-hoc and depends on a supervised model trained on structural similarity labels (computed via ms_compute_structural_similarity.py over GNPS/MassBank data); it cannot run model-free
- The explanation targets query–reference spectrum pairs identified via a pairs id file; if --pairs is empty, every query is paired with every reference, which is computationally expensive at library scale
- The article's discussion provides no quantitative metrics or confidence intervals in the extracted text for the comparative performance claims against Cosine, Modified Cosine, Spec2Vec, and MS2DeepScore

## Evidence

- [intro] The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity: "The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity"
- [abstract] TransExION detects related fragments between MS/MS spectra through their mass difference: "TransExION detects related fragments between MS/MS spectra through their mass difference"
- [discussion] The explanation of query product ions is achieved based on the contribution of query-reference product ion associations to spectral similarity prediction: "The explanation of query product ions is achieved based on the contribution of query-reference product ion associations to spectral similarity prediction"
- [readme] It is a heatmap indicating the association between the query fragments and reference fragments.: "It is a heatmap indicating the association between the query fragments and reference fragments."
- [readme] python ms_explain_spectral_similarity.py --db [reference spectrum file] --query [query spectrum file] --pairs [spectrum pair id file] --model [model file] --output [output folder]: "python ms_explain_spectral_similarity.py --db [reference spectrum file] --query [query spectrum file] --pairs [spectrum pair id file] --model [model file] --output [output folder]"
