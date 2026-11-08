---
name: spectral-pair-annotation-for-structure-elucidation
description: Use when a query MS/MS spectrum has been matched to a reference spectrum
  of an analogue and you need to explain the similarity score by linking query product
  ions to reference product ions via their mass differences.
when_to_use_negative:
- You only need a scalar spectral similarity score without per-fragment explanation
  — use ms_test_spectral_similarity.py instead
- No trained TransExION model or structural-similarity-trained checkpoint is available;
  the post-hoc relevance depends on the trained transformer weights
- The spectra are not in (or convertible from MGF to) the h5py format the TransExION
  pipeline requires
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TransExION
  role: Transformer-based spectral similarity model whose post-hoc explanation module
    (layer-wise relevance propagation) produces the query-reference product ion relevance
    heatmaps
  repo: github.com/banhdzui/TransExION
- name: PyTorch
  role: Framework in which the TransExION model and its explanation scripts are implemented
    and run
  repo: github.com/banhdzui/TransExION
- name: h5py
  role: Format for spectra databases and the spectrum pair id file consumed by the
    explanation script
  repo: github.com/banhdzui/TransExION
- name: Matplotlib
  role: Rendering of the relevance matrices as heatmap images
  repo: github.com/banhdzui/TransExION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:6fc0ac79cefa742f21661d5bcae6dd416e2e883a3f2faa100e90fc37931fb012
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matplotlib
  - PyTorch
  - TransExION
  - h5py
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Spectral pair annotation for structure elucidation

## Summary

Uses TransExION's post-hoc explanation module (layer-wise relevance propagation) to compute pairwise relevance between query and reference product ions, producing heatmaps that reveal mass-difference fragment associations supporting structure elucidation of unknown analogues.

## When to use

Use when a query MS/MS spectrum has been matched to a reference spectrum of an analogue and you need to explain the similarity score by linking query product ions to reference product ions via their mass differences.

## When NOT to use

- You only need a scalar spectral similarity score without per-fragment explanation — use ms_test_spectral_similarity.py instead
- No trained TransExION model or structural-similarity-trained checkpoint is available; the post-hoc relevance depends on the trained transformer weights
- The spectra are not in (or convertible from MGF to) the h5py format the TransExION pipeline requires

## Inputs

- Trained TransExION model file (e.g. GNPS_MassBank.ms.model)
- Reference spectrum library in h5py format (e.g. GNPS_MassBank_train.db)
- Query spectra in h5py format (e.g. GNPS_MassBank_test.db)
- Spectrum pair id file in h5py format (converted from csv via ms_covert_csv2h5py.py) specifying the query-reference pairs to explain

## Outputs

- Relevance heatmap image files, one per spectrum pair, written to the output directory
- Pairwise relevance matrix between query and reference product ions (top q=20 query ions x top k=3 reference ions each) with annotated mass differences

## How to apply

Load a trained TransExION model (e.g. GNPS_MassBank.ms.model) together with reference and query spectra in h5py format (e.g. GNPS_MassBank_train.db, GNPS_MassBank_test.db) and convert the query/reference spectrum identifiers into a spectrum-pairs h5py file using ms_covert_csv2h5py.py. Run ms_explain_spectral_similarity.py on these pairs, which applies layer-wise relevance propagation through the transformer to estimate the pairwise relevance between every query and reference product ion. Rank the top q=20 query product ions by overall contribution and retain the top k=3 most associated reference product ions for each query ion. Render the relevance matrix as a heatmap where each cell shows the query product ion, reference product ion(s) and their mass difference (e.g. +15.99 Da for DL-beta-Homophenylalanine, -14.02 Da for Scopularide F), which reveals substructure links such as an oxygen difference between reference and query elemental compositions.

## Related tools

- **TransExION** (Transformer-based spectral similarity model whose post-hoc explanation module (layer-wise relevance propagation) produces the query-reference product ion relevance heatmaps) — github.com/banhdzui/TransExION
- **PyTorch** (Framework in which the TransExION model and its explanation scripts are implemented and run) — github.com/banhdzui/TransExION
- **h5py** (Format for spectra databases and the spectrum pair id file consumed by the explanation script) — github.com/banhdzui/TransExION
- **Matplotlib** (Rendering of the relevance matrices as heatmap images) — github.com/banhdzui/TransExION

## Examples

```
python ms_explain_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --pairs spectrum_pairs.db --model GNPS_MassBank.ms.model --output heatmaps/
```

## Evaluation signals

- Each generated heatmap file exists in the output directory and is a non-empty image
- The DL-beta-Homophenylalanine heatmap reproduces the +15.99 Da offset associations reported in Fig. 8 (e.g. m/z 163.06 -> 179.03, 145.06 -> 161.05, 120.06 -> 136.04)
- The Scopularide F heatmap reproduces the -14.02 Da offset associations reported in Fig. 9 (e.g. m/z 157.13 -> 171.15, 351.26 -> 365.29)
- The retained associations correspond to the top q=20 query ions by overall contribution with at most k=3 reference ions each
- Mass differences annotated in heatmap cells are consistent with the stated elemental-composition differences between query and reference structures

## Limitations

- The explanation is post-hoc and supports, but does not by itself prove, structure elucidation; it links query to reference product ions to assign substructures of the reference compound
- Pairs must be pre-selected in a spectrum pair id file; without a --pairs file every query is paired with every reference spectrum
- Requires spectra and pairs in h5py format converted from MGF/csv with the provided conversion scripts

## Evidence

- [readme] post-hoc explanation heatmap: "It is a heatmap indicating the association between the query fragments and reference fragments."
- [readme] heatmap cell content: "Each heatmap cell shows the query product ion (the first value in the brackets), reference product ion(s) and their mass difference (the second value in the brackets)."
- [readme] explanation script usage: "The post-hoc explanation is demonstrated in the script ```ms_explain_spectral_similarity.py```."
- [readme] explanation output description: "the parameter ```--output``` is the path to a directory where contains generated heatmap files for every pair."
- [discussion] article claim on post-hoc module: "the TransExION framework is built with a unique post hoc explanation module"
- [discussion] article claim on linking fragments: "the post hoc explanation module can provide a good starting point of the structure elucidation of unknowns by linking query with reference product ions"
- [discussion] article claim on functional group modifications: "the TransExION model can explain simultaneous modifications of distinct functional groups from the reference compound"
