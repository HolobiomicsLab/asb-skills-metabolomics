---
name: supervised-siamese-spectral-embedding-training
description: Use when building a spectral similarity predictor from annotated tandem
  MS libraries (e.g. GNPS ALL_GNPS.mgf) where spectral similarity must proxy chemical
  structural similarity, and no test structure may leak into training.
when_to_use_negative:
- You only need to run inference on queries with an existing pretrained MS2Query library
  — download the pretrained models instead of retraining (e.g. Zenodo record 6124552
  for positive mode, 7104184 for negative mode)
- Input spectra are unannotated query MS/MS from a metabolomics run — training requires
  fully annotated library spectra with structural labels (InChIKey)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2Deepscore
  role: Supervised siamese model trained on annotated GNPS spectra with pairs labeled
    by chemical structural similarity
  repo: https://github.com/iomega/ms2query
- name: Spec2Vec
  role: Unsupervised embedding model trained on all library spectra for 30 epochs
    with peaks binned to 2 decimals
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Library cleaning and intensity normalization before training (peak filtering,
    metadata cleaning, InChIKey completion via PubChem)
- name: scikit-learn
  role: Random forest re-ranker (250 estimators, max depth 5, MSE loss) trained on
    the same structure-disjoint splits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:affc9a16fe77a06ae7dd624a70ad0b404b618acd593595fad6260b7a9a1629bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - Spec2Vec
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# supervised-siamese-spectral-embedding-training

## Summary

Train a supervised siamese spectral embedding model (MS2Deepscore) on fully annotated GNPS library spectra, using spectrum pairs labeled by chemical structural similarity, to produce embeddings whose similarities act as proxies for chemical similarity in analogue search.

## When to use

Use when building a spectral similarity predictor from annotated tandem MS libraries (e.g. GNPS ALL_GNPS.mgf) where spectral similarity must proxy chemical structural similarity, and no test structure may leak into training.

## When NOT to use

- You only need to run inference on queries with an existing pretrained MS2Query library — download the pretrained models instead of retraining (e.g. Zenodo record 6124552 for positive mode, 7104184 for negative mode)
- Input spectra are unannotated query MS/MS from a metabolomics run — training requires fully annotated library spectra with structural labels (InChIKey)

## Inputs

- GNPS ALL_GNPS.mgf annotated mass spectral library (downloaded e.g. 15 Nov 2021 from https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.mgf)
- Spectra with InChIKey/compound structure annotations for structural-similarity labeling and structure-disjoint splitting
- PubChem lookups (via matchms extras) to fill missing InChIKeys from compound name and molecular formula

## Outputs

- Trained MS2Deepscore siamese model file
- Trained Spec2Vec model (30 epochs, peaks binned to 2 decimal places)
- Training-data-split manifest recording structure-disjoint train/test assignment
- Pre-calculated mass spectral embeddings for all library spectra

## How to apply

First clean the GNPS library (e.g. ALL_GNPS.mgf) with matchms: remove spectra with fewer than 3 peaks, remove peaks above 1000 Da and peaks below 0.1% of the highest peak intensity, cap spectra at 500 peaks by dropping lowest-intensity peaks, and normalize intensities. Create structure-disjoint train/test splits based on InChIKey compound structure so no test spectrum's structure appears in training, and record the split in a manifest. Train the MS2Deepscore siamese model on the fully annotated GNPS spectra with spectrum pairs labeled by chemical structural similarity; separately train Spec2Vec unsupervised for 30 epochs with fragment peaks binned to 2 decimals. For benchmarking, ensure none of the test spectra were used for training MS2Deepscore, Spec2Vec, or the downstream random forest (250 estimators, max depth 5, MSE loss), preventing data leakage between models.

## Related tools

- **MS2Deepscore** (Supervised siamese model trained on annotated GNPS spectra with pairs labeled by chemical structural similarity) — https://github.com/iomega/ms2query
- **Spec2Vec** (Unsupervised embedding model trained on all library spectra for 30 epochs with peaks binned to 2 decimals) — https://github.com/iomega/ms2query
- **matchms** (Library cleaning and intensity normalization before training (peak filtering, metadata cleaning, InChIKey completion via PubChem))
- **scikit-learn** (Random forest re-ranker (250 estimators, max depth 5, MSE loss) trained on the same structure-disjoint splits)

## Examples

```
conda create --name ms2query python=3.9; conda activate ms2query; pip install ms2query
```

## Evaluation signals

- Structure-disjoint split verified: no InChIKey/2D structure present in both train and analogue-search test set (recorded in split manifest)
- No test spectrum was used for training MS2Deepscore, Spec2Vec, or MS2Query — confirm by cross-checking the manifest against training spectra lists
- Trained model reproduces MS2Deepscore spectral similarities that correlate with chemical structural similarity on held-out pairs
- Model files, Spec2Vec model, random forest, and split manifest are all saved together and loadable by the ms2query package
- Benchmarking MS2Query on the held-out test set outperforms cosine score, modified cosine score, and MS2Deepscore alone for exact matches and analogues

## Limitations

- Performance may differ on newly generated data: 'it is hard to objectively judge if MS2Query performs similarly on newly generated data' compared to the benchmarking test set
- Library cleaning steps (peak filtering, 1000 Da cap, 0.1% relative-intensity floor, 500-peak cap) constrain which spectra can be used for training
- Missing InChIKeys must be filled by PubChem name/formula lookup, which may fail for unnamed or novel compounds

## Evidence

- [results] Structure-disjoint splits prevent leakage: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
- [methods] Siamese training on annotated spectra with structural similarity labels: [evidence span withheld — anchor status: not_found]
- [results] Library cleaning with matchms: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] Dataset cleaning workflow: "The dataset was first cleaned using matchms33. The metadata was cleaned to get a uniform format and to remove or correct misplaced metadata."
- [intro] Spectral similarity as proxy for chemical similarity: "To perform a succesful analogue search, it is important to have a spectral similarity score that serves as a good proxy for chemical similarity even if two molecules are not identical."
- [discussion] Precomputed embeddings for library spectra: "The biggest speed improvement was achieved by pre-calculating mass spectral embeddings for Spec2Vec and MS2Deepscore."
- [discussion] Random forest re-ranking on preselected candidates: "First, potential candidates are selected based on MS2Deepscore, followed by re-ranking the spectra by using a random forest model."
- [discussion] Models and raw data archived on Zenodo: "The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540."
