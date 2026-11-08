---
name: spectral-library-analogue-search
description: Use when an unknown MS/MS spectrum must be searched against a spectral
  library (e.g. mergedGNPS) to find structural analogues, and heuristic scores like
  Cosine or Modified Cosine are insufficient at high Tanimoto cut-offs.
when_to_use_negative:
- When a direct, confident spectral-library identification (match to the same compound,
  not an analogue) already exists for the query spectrum
- When only raw spectra are available without MGF/h5py conversion and no preprocessing
  pipeline is in place
- When the goal is de novo structure generation from MS/MS rather than library-based
  analogue retrieval
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TransExION
  role: Transformer-based supervised model predicting spectral similarity correlated
    with structural similarity; used for library analogue search and explanation
  repo: https://github.com/banhdzui/TransExION
- name: Cosine similarity
  role: Baseline heuristic spectral similarity measure outperformed by TransExION
    in analogue retrieval
- name: Spec2Vec
  role: Unsupervised baseline spectral similarity method compared against TransExION
- name: MS2DeepScore
  role: Deep-learning baseline similarity measure outperformed by TransExION, especially
    at high Tanimoto cut-offs
- name: meRgeION
  role: Consumes TransExION similarity scores for molecular networking and analogue
    annotation of network nodes
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:dca60a9ade25b0bc5e839152f8f557abc374daf21a68643d2b72c9998a9fcc0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - Cosine similarity
  - MS2DeepScore
  - Matplotlib
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - meRgeION
  - pyteomics
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# spectral-library-analogue-search

## Summary

Use a pre-trained TransExION Transformer model to estimate spectral similarity between query MS/MS spectra and spectral library references, retrieving structural analogues whose predicted similarity reflects Tanimoto structural similarity.

## When to use

Use when an unknown MS/MS spectrum must be searched against a spectral library (e.g. mergedGNPS) to find structural analogues, and heuristic scores like Cosine or Modified Cosine are insufficient at high Tanimoto cut-offs.

## When NOT to use

- When a direct, confident spectral-library identification (match to the same compound, not an analogue) already exists for the query spectrum
- When only raw spectra are available without MGF/h5py conversion and no preprocessing pipeline is in place
- When the goal is de novo structure generation from MS/MS rather than library-based analogue retrieval

## Inputs

- Query MS/MS spectra in MGF format (converted to h5py)
- Reference spectral library (e.g. mergedGNPS, GNPS-derived) in MGF/h5py format
- Pre-trained TransExION model file (Zenodo record 8175528)
- Optional spectrum pair id file in h5py format to restrict scored pairs

## Outputs

- Predicted spectral similarity scores per query-reference pair (CSV result file)
- Ranked list of structural analogue hits per query spectrum
- Optional post-hoc explanation heatmaps of query-reference fragment associations per pair

## How to apply

Convert both reference (library) and query spectra from MGF to h5py format with ms_convert_mgf2h5py.py. Download the pre-trained model trained on GNPS-derived data from Zenodo (record 8175528) or retrain with ms_train_spectral_similarity.py on GNPS_MassBank train/val splits. Run ms_test_spectral_similarity.py pairing every query spectrum against every reference spectrum (leave --pairs empty for all-pairs). Rank reference hits by predicted spectral similarity and retain top hits above a 0.6 similarity threshold (e.g. top three analogues per node in the meRgeION analogue-search workflow). Optionally run ms_explain_spectral_similarity.py to generate heatmaps linking query product ions to reference product ions for substructure assignment. TransExION additionally considers the mass difference between query and reference fragments, which explains its advantage over Cosine, Modified Cosine, Spec2Vec and MS2DeepScore, especially when nearly-identical structure matching is desired.

## Related tools

- **TransExION** (Transformer-based supervised model predicting spectral similarity correlated with structural similarity; used for library analogue search and explanation) — https://github.com/banhdzui/TransExION
- **Cosine similarity** (Baseline heuristic spectral similarity measure outperformed by TransExION in analogue retrieval)
- **Spec2Vec** (Unsupervised baseline spectral similarity method compared against TransExION)
- **MS2DeepScore** (Deep-learning baseline similarity measure outperformed by TransExION, especially at high Tanimoto cut-offs)
- **meRgeION** (Consumes TransExION similarity scores for molecular networking and analogue annotation of network nodes)

## Examples

```
python ms_test_spectral_similarity.py --db mergedGNPS_library.db --query query_spectra.db --model GNPS_MassBank.ms.model --output analogue_search_results.csv
```

## Evaluation signals

- Predicted similarities should correlate strongly with Tanimoto scores (r = 0.811 reported on mergedGNPS), with low squared error and minimal variation across 10 equal-width Tanimoto bins
- Retrieval should outperform Cosine, Modified Cosine, Spec2Vec and MS2DeepScore baselines, particularly at high Tanimoto distance cut-offs
- In molecular networking, filtering pairwise scores > 0.6 should recover all seven SNAP-MS compound-family sub-networks from the 925-sample bacteria extract library (Fig. 10 topology)
- Explanation heatmaps should show query product ion / reference product ion / mass-difference associations consistent with substructure links of the retrieved analogue
- Output CSV from ms_test_spectral_similarity.py should contain one similarity score per scored pair (all query-reference pairs if --pairs is empty)

## Limitations

- TransExION is a supervised model trained on GNPS-derived MS/MS library data; performance depends on the query spectra resembling the training domain
- The version of the model, code environment and dependencies are not fully reported; training data and pretrained weights must be obtained from the Zenodo record (8175528)
- The Tanimoto distance cut-off defining 'nearly-identical structure matching' is not explicitly specified in the paper

## Evidence

- [discussion] TransExION outperforms Cosine, Modified Cosine, Spec2Vec and MS2DeepScore in retrieving structure analogs, especially at high Tanimoto distance cut-offs.: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [abstract] TransExION detects related fragments via mass difference: "TransExION detects related fragments between MS/MS spectra through their mass difference"
- [readme] Testing script computes spectral similarity with all-pairs default: "To compute spectral similarity between spectra using a trained TransExION model, the script ```ms_test_spectral_similarity.py``` can be used with parameters"
- [readme] All-pairs behaviour when --pairs is empty: "If the parameter ```--pairs``` is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity."
- [readme] Explanation module generates heatmaps of query-reference fragment associations: "It is a heatmap indicating the association between the query fragments and reference fragments."
- [readme] Training data availability on Zenodo: "The data we used to train the model can be found in [zenodo link](https://zenodo.org/record/8175528)."
