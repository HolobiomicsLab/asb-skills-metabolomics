---
name: pairwise-spectral-similarity-scoring
description: Use when a set of MS/MS feature spectra (e.g. .mgf with feature_id, precursor
  m/z, m/z-intensity tuples) must be converted into pairwise similarity matrices to
  drive downstream embedding, network, or heatmap visualizations.
when_to_use_negative:
- The input is already a processed similarity matrix or a specXplore session data
  object — rerunning the importing pipeline would redundantly recompute MS2Query and
  pairwise scores
- The spectra/metabolite classes are poorly covered by or unrelated to ms2deepscore's
  training data, since the pretrained model can only be expected to work well for
  spectra well covered by or related to its training data
- The dataset far exceeds ~5000 features after processing, where the article anticipates
  the dashboard and settings will face difficulties
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Imports the .mgf spectral data into Python, performs basic specXplore preprocessing,
    and can add/rename the 'feature_id' metadata key via matchms.Spectrum
- name: ms2deepscore
  role: Provides the primary machine-learned pairwise spectral similarity scores that
    serve as specXplore's quantitative backbone
- name: MS2Query
  role: Supplies the pretrained models (and local library search) used to generate
    the ms2deepscore machine-learning similarities
- name: spec2vec
  role: One of the two auxiliary similarity scores computed automatically alongside
    ms2deepscore and modified cosine by the importing pipeline
- name: specXplore
  role: Python package whose importing pipeline orchestrates the automatic computation
    of the three pairwise similarity matrices and produces the session data object
  repo: https://github.com/kevinmildau/specXplore
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:53f1bf62901403a3b55386d4376512d648c24201425568549603914e9cc14341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - plotly
  - spec2vec
  - specXplore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pairwise-spectral-similarity-scoring

## Summary

Compute the full pairwise spectral similarity matrices (ms2deepscore as primary, plus modified cosine and spec2vec) for LC-MS/MS feature spectra in .mgf format via the specXplore importing pipeline; these matrices are the quantitative backbone for t-SNE overviews, network views, and similarity heatmaps.

## When to use

Use when a set of MS/MS feature spectra (e.g. .mgf with feature_id, precursor m/z, m/z-intensity tuples) must be converted into pairwise similarity matrices to drive downstream embedding, network, or heatmap visualizations.

## When NOT to use

- The input is already a processed similarity matrix or a specXplore session data object — rerunning the importing pipeline would redundantly recompute MS2Query and pairwise scores
- The spectra/metabolite classes are poorly covered by or unrelated to ms2deepscore's training data, since the pretrained model can only be expected to work well for spectra well covered by or related to its training data
- The dataset far exceeds ~5000 features after processing, where the article anticipates the dashboard and settings will face difficulties

## Inputs

- .mgf file with MS/MS feature spectra, each entry containing a unique feature identifier under the key 'feature_id', precursor m/z, and m/z-intensity tuples
- MS2Query pretrained model and library files for positive mode (Zenodo record 10527997) or negative mode (Zenodo record 10528030), stored in separate folders ('pos' sub-directory for the illustrative-examples notebooks)

## Outputs

- ms2deepscore pairwise similarity matrix (primary score) for all features
- modified cosine and spec2vec pairwise similarity matrices for the same feature set
- specXplore session data object saved to the hard drive, loadable into a specXplore dashboard session

## How to apply

Import the .mgf spectral data into Python using matchms and perform basic specXplore preprocessing, ensuring every spectrum carries a 'feature_id' metadata key (rename wrong keys via matchms.Spectrum or a text editor). Feed the preprocessed spectra into the specXplore importing pipeline (Jupyter notebook based, e.g. demo.ipynb), which automatically computes all pairwise similarities with three scores: ms2deepscore, modified cosine, and spec2vec. For the machine-learned ms2deepscore score, use the pretrained models shipped with MS2Query, placing the appropriate positive-mode or negative-mode model/library files (e.g. from Zenodo records 10527997 / 10528030) into separate model folders. Treat ms2deepscore as the primary score for downstream t-SNE embedding and thresholding, and retain modified cosine and spec2vec matrices for comparative overlays (e.g. implied-adjacency markers on augmap heatmaps). The pipeline returns a specXplore session data object saved to disk that can be loaded directly into the dashboard. Note the article's expectation that MS2Query dominates runtime (≈62 min of <75 min for the <4000-feature urine metabolome dataset) while all pairwise similarity computations took ≈9 min.

## Related tools

- **matchms** (Imports the .mgf spectral data into Python, performs basic specXplore preprocessing, and can add/rename the 'feature_id' metadata key via matchms.Spectrum)
- **ms2deepscore** (Provides the primary machine-learned pairwise spectral similarity scores that serve as specXplore's quantitative backbone)
- **MS2Query** (Supplies the pretrained models (and local library search) used to generate the ms2deepscore machine-learning similarities)
- **spec2vec** (One of the two auxiliary similarity scores computed automatically alongside ms2deepscore and modified cosine by the importing pipeline)
- **specXplore** (Python package whose importing pipeline orchestrates the automatic computation of the three pairwise similarity matrices and produces the session data object) — https://github.com/kevinmildau/specXplore

## Examples

```
pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- All three similarity matrices (ms2deepscore, modified cosine, spec2vec) exist and are complete for every feature pair in the .mgf file
- Every spectrum carries the 'feature_id' key; the .mgf entries include unique feature identifiers, precursor m/z, and m/z-intensity tuples
- Runtime scaling matches the article: for the <4000-feature urine metabolome dataset, all pairwise spectral similarity computations complete in roughly 9 min (with ≈62 min for MS2Query)
- The output is a well-formed specXplore session data object that loads directly into the dashboard without a second preprocessing pass
- The correct positive- or negative-mode MS2Query pretrained model files were placed in the models folder and used by the pipeline

## Limitations

- On macOS arm64 systems, ms2deepscore package versions may produce similarity predictions that are not in accordance with results on other systems (issue 199) — with no error or warning — making default ms2deepscore scores unreliable; the model files and predictions work on other systems
- ms2deepscore applicability is limited to spectra/metabolites covered by or related to its training data; no coverage check was performed
- Scalability beyond ~5000 features is untested; no software version numbers are stated for any of the tools
- The current version of specXplore works on macOS and Linux but fails on Windows (a Windows installation path via Anaconda and MS Visual C++ Redistributable is described)

## Evidence

- [methods] Automated triple-score computation: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [methods] Pretrained models from MS2Query: "For the machine learning scores, pretrained models provided with MS2Query are used."
- [methods] Import via matchms: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [results] Reported runtime for pairwise similarity computations: "approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [readme] macOS arm64 reliability warning: [evidence span withheld — anchor status: external_doc]
- [readme] feature_id key requirement: [evidence span withheld — anchor status: external_doc]
