---
name: heatmap-relevance-matrix-visualization
description: Use when a trained TransExION model has scored a query-reference MS/MS
  spectrum pair and you must explain WHICH product-ion links drove the similarity,
  e.g. to trace an oxygen (+15.99 Da) or CH2 (-14.02 Da) modification between analogues.
when_to_use_negative:
- You only need numeric spectral-similarity scores for many pairs — use ms_test_spectral_similarity.py
  instead; explanation heatmaps are per-pair and computationally heavier
- The query and reference spectra have no plausible structural relation (no shared
  substructures), since the heatmap links fragments via substructure/mass-difference
  associations rather than full-spectrum matches
- Input spectra are in MGF format without conversion — the scripts require h5py .db
  files produced by ms_convert_mgf2h5py.py
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: TransExION
  role: Trained Transformer model whose layer-wise relevance propagation output is
    rendered as the relevance-matrix heatmap
  repo: https://github.com/banhdzui/TransExION
- name: PyTorch
  role: Framework in which the model and the post-hoc explanation (LRP) are implemented
  repo: https://github.com/banhdzui/TransExION
- name: Matplotlib
  role: Renders the relevance matrices as heatmap image files per spectrum pair
- name: h5py
  role: Format for the reference/query spectrum databases and the spectrum-pairs id
    file
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:458b4dae49e7bfe5da34c09b1397e96ea523d1ce35079c4749a9c4fb03f6770b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - Matplotlib
  - PyTorch
  - TransExION
  - h5py
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# heatmap-relevance-matrix-visualization

## Summary

Generates post-hoc relevance-matrix heatmaps from TransExION that visualize the contribution of each query-reference product-ion association to the predicted MS/MS spectral similarity, revealing mass-difference relationships (e.g. +15.99 Da, -14.02 Da) that support structure elucidation of unknowns.

## When to use

Use when a trained TransExION model has scored a query-reference MS/MS spectrum pair and you must explain WHICH product-ion links drove the similarity, e.g. to trace an oxygen (+15.99 Da) or CH2 (-14.02 Da) modification between analogues.

## When NOT to use

- You only need numeric spectral-similarity scores for many pairs — use ms_test_spectral_similarity.py instead; explanation heatmaps are per-pair and computationally heavier
- The query and reference spectra have no plausible structural relation (no shared substructures), since the heatmap links fragments via substructure/mass-difference associations rather than full-spectrum matches
- Input spectra are in MGF format without conversion — the scripts require h5py .db files produced by ms_convert_mgf2h5py.py

## Inputs

- Pre-trained TransExION model file (e.g. GNPS_MassBank.ms.model)
- Reference/library MS/MS spectra in h5py format (e.g. GNPS_MassBank_train.db)
- Query MS/MS spectra in h5py format (e.g. GNPS_MassBank_test.db)
- Spectrum-pair id file in h5py format (converted from CSV via ms_covert_csv2h5py.py) listing the pairs to explain

## Outputs

- One heatmap image file per spectrum pair in the output directory, each showing query x reference product-ion relevance with mass differences per cell
- Implicit pairwise relevance matrix per pair (top q=20 query ions, top k=3 associated reference ions per query ion)

## How to apply

Load the pre-trained TransExION model (e.g. GNPS_MassBank.ms.model) together with reference and query spectra in h5py format (GNPS_MassBank_train.db, GNPS_MassBank_test.db), and build a spectrum-pairs id file containing only the pairs of interest using ms_covert_csv2h5py.py. Run ms_explain_spectral_similarity.py, which computes pairwise relevance between query and reference product ions via layer-wise relevance propagation and writes one heatmap file per pair to the output directory. For each pair, rank the top q=20 query product ions by overall contribution and retain the top k=3 most associated reference product ions per query ion to keep the matrix readable. Render each relevance matrix as a heatmap where cells show the query product ion (first value in brackets), reference product ion(s), and their mass difference (second value in brackets). Validate by confirming each heatmap file is a non-empty image and that the expected mass-difference offsets appear (e.g. m/z 163.06 -> 179.03, 145.06 -> 161.05, 120.06 -> 136.04 for the +15.99 Da shift in DL-beta-Homophenylalanine; 157.13 -> 171.15, 351.26 -> 365.29 for the -14.02 Da shift in Scopularide F), reproducing Figs. 8 and 9 of the paper.

## Related tools

- **TransExION** (Trained Transformer model whose layer-wise relevance propagation output is rendered as the relevance-matrix heatmap) — https://github.com/banhdzui/TransExION
- **PyTorch** (Framework in which the model and the post-hoc explanation (LRP) are implemented) — https://github.com/banhdzui/TransExION
- **Matplotlib** (Renders the relevance matrices as heatmap image files per spectrum pair)
- **h5py** (Format for the reference/query spectrum databases and the spectrum-pairs id file)

## Examples

```
python ms_explain_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --pairs spectrum_pairs.db --model GNPS_MassBank.ms.model --output heatmaps/
```

## Evaluation signals

- Each generated heatmap file exists in the output directory and is a non-empty image
- The DL-beta-Homophenylalanine heatmap shows the +15.99 Da offset between query and reference product ions (e.g. 163.06 -> 179.03, 145.06 -> 161.05, 120.06 -> 136.04), consistent with one extra oxygen in the reference structure
- The Scopularide F (cyclic lipodepsipeptide) heatmap shows the -14.02 Da offset (e.g. 157.13 -> 171.15, 351.26 -> 365.29), reproducing Figs. 8 and 9 of the paper
- Only the top q=20 query product ions by overall contribution are retained, with at most k=3 most-associated reference product ions per query ion
- The pairs file contains exactly the two case-study pairs (splash10-00xr-0900000000-e86eefea78d902b2e731 and splash10-0udi-2014761900-e58065981f99435865cd) and two heatmap files are produced

## Limitations

- The explanation is post-hoc and depends on the trained model; heatmap quality reflects whatever data (GNPS/MassBank) the model was trained on
- Only the top q=20 query ions and k=3 reference ions per query ion are shown — lower-contributing fragment associations are not visualized
- Details such as version, code environment, or dependencies of the post-hoc module are not fully documented in the article's discussion

## Evidence

- [discussion] The explanation of query product ions is achieved based on the contribution of query-reference product ion associations to spectral similarity prediction: "The explanation of query product ions is achieved based on the contribution of query-reference product ion associations to spectral similarity prediction"
- [readme] It is a heatmap indicating the association between the query fragments and reference fragments.: "It is a heatmap indicating the association between the query fragments and reference fragments."
- [readme] Each heatmap cell shows the query product ion (the first value in the brackets), reference product ion(s) and their mass difference (the second value in the brackets).: "Each heatmap cell shows the query product ion (the first value in the brackets), reference product ion(s) and their mass difference (the second value in the brackets)."
- [discussion] the post hoc explanation module can provide a good starting point of the structure elucidation of unknowns by linking query with reference product ions: "the post hoc explanation module can provide a good starting point of the structure elucidation of unknowns by linking query with reference product ions"
- [readme] the parameter ```--output``` is the path to a directory where contains generated heatmap files for every pair.: "the parameter ```--output``` is the path to a directory where contains generated heatmap files for every pair."
