---
name: msms-feature-preprocessing
description: Use when a .mgf file of MS/MS feature spectra (e.g. wheat or urine metabolome
  LC–MS/MS with <1000–<4000 features) must be turned into a specXplore session data
  object with pairwise similarity matrices before t-SNE or dashboard exploration.
when_to_use_negative:
- Input data is not in .mgf format and has not been exported via vendor software or
  MZmine3's GNPS-FBMN-IIN export — specXplore currently requires .mgf with a 'feature_id'
  key
- Dataset substantially exceeds ~5000 features — the article reports scalability is
  untested beyond this and difficulties are anticipated for larger data sets
- A Windows-only environment is the target — the README states the current version
  fails on Windows, and macOS arm64 systems are affected by ms2deepscore issue 199,
  silently producing unreliable similarity predictions
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Python package whose importing pipeline performs basic preprocessing and builds
    the session data object; installed from GitHub, MIT licensed
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports the .mgf spectra into Python and provides the Spectrum module for
    renaming/adding the 'feature_id' metadata key
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: Provides the pretrained machine-learning models used in the pipeline; its
    local library search dominates preprocessing time (~62 min on the urine data)
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Computes the primary pairwise spectral similarity scores consumed downstream
    by the t-SNE embedding
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:30158e2300f38d875b2141532f2fd5d976fd5772a2820f8a2b564cae232f1292
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# msms-feature-preprocessing

## Summary

Preprocess LC–MS/MS spectral feature data with the specXplore importing pipeline: load .mgf spectra via matchms, ensure 'feature_id' metadata keys, and automatically compute pairwise ms2deepscore, modified cosine, and spec2vec similarities into a saved session data object for dashboard exploration.

## When to use

Use when a .mgf file of MS/MS feature spectra (e.g. wheat or urine metabolome LC–MS/MS with <1000–<4000 features) must be turned into a specXplore session data object with pairwise similarity matrices before t-SNE or dashboard exploration.

## When NOT to use

- Input data is not in .mgf format and has not been exported via vendor software or MZmine3's GNPS-FBMN-IIN export — specXplore currently requires .mgf with a 'feature_id' key
- Dataset substantially exceeds ~5000 features — the article reports scalability is untested beyond this and difficulties are anticipated for larger data sets
- A Windows-only environment is the target — the README states the current version fails on Windows, and macOS arm64 systems are affected by ms2deepscore issue 199, silently producing unreliable similarity predictions

## Inputs

- MS/MS spectral feature data in .mgf format (each entry with a unique feature identifier under key 'feature_id', precursor m/z, and m/z–intensity tuples)
- Pretrained MS2Query model and library files for positive or negative ion mode (Zenodo records 10527997 and 10528030), organized in pos/neg subfolders

## Outputs

- specXplore session data object saved to the hard drive, consumable directly by the specXplore dashboard
- Pairwise spectral similarity matrices for ms2deepscore, modified cosine, and spec2vec
- t-SNE embedding of the ms2deepscore pairwise similarity matrix (e.g. urine_tsne_embedding.csv with feature_id, x, y columns)

## How to apply

Install specXplore in a conda environment with Python 3.8 (pip install 'git+https://github.com/kevinmildau/specXplore.git') on macOS or Linux, and place the pretrained MS2Query model/library files for positive or negative mode (Zenodo records 10527997 / 10528030) in separate folders required by the importing pipeline. Input must be a .mgf file whose spectra carry a feature identifier under the key 'feature_id'; rename wrong keys using the matchms.Spectrum module or a text-editor replacement of 'wrong_key=' to 'feature_id='. Run the preprocessing in Jupyter (demo.ipynb or the example-feature-id-processing notebook), replacing the demo data path with your .mgf file: matchms imports the spectra, basic specXplore processing is performed, and the pipeline automatically computes pairwise similarities using ms2deepscore (primary score), modified cosine, and spec2vec. The resulting specXplore session data object is saved to the hard drive and can be fed directly into a specXplore dashboard session. Expect roughly <15 min preprocessing for the wheat data (<1000 features) and <75 min for the urine metabolome data (<4000 features), of which ~62 min is MS2Query library search and ~9 min all pairwise spectral similarity computations.

## Related tools

- **specXplore** (Python package whose importing pipeline performs basic preprocessing and builds the session data object; installed from GitHub, MIT licensed) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports the .mgf spectra into Python and provides the Spectrum module for renaming/adding the 'feature_id' metadata key) — https://github.com/kevinmildau/specXplore
- **MS2Query** (Provides the pretrained machine-learning models used in the pipeline; its local library search dominates preprocessing time (~62 min on the urine data)) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Computes the primary pairwise spectral similarity scores consumed downstream by the t-SNE embedding)

## Examples

```
conda create --name specxplore_environment python=3.8 && conda activate specxplore_environment && pip install "git+https://github.com/kevinmildau/specXplore.git" && jupyter-notebook notebooks/demo.ipynb
```

## Evaluation signals

- The pipeline completes and writes a specXplore session data object to disk that a dashboard session can load without further transformation
- All three similarity scores (ms2deepscore, modified cosine, spec2vec) are present in the computed pairwise similarity output, not just a single score
- Every spectrum retains a feature identifier under the key 'feature_id' after preprocessing; no spectra are lost relative to the input .mgf
- For a reference-scale dataset, runtime matches reported expectations (<15 min for wheat <1000 features; <75 min for urine <4000 features, ~9 min for all pairwise similarity computations)
- The t-SNE output CSV validates: exists, contains feature_id/x/y fields with finite coordinates, and high-ms2deepscore-similarity features occupy nearby positions

## Limitations

- specXplore works on macOS and Linux but fails on Windows; on macOS arm64, ms2deepscore issue 199 yields similarity predictions that silently deviate from other systems with no error or warning
- Scalability beyond ~5000 features is untested; the dashboard is expected to face difficulties for larger data sets with liberal settings
- ms2deepscore can only be expected to work well for spectra and metabolites covered by or related to its training data; no coverage check is reported
- No tool version numbers are stated for any software involved

## Evidence

- [methods] Spectral data are imported into Python using matchms, and basic specXplore data processing is performed: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [methods] Three similarity scores computed automatically: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [readme] mgf requirement and feature_id key: [evidence span withheld — anchor status: external_doc]
- [readme] feature_id key expectation: [evidence span withheld — anchor status: external_doc]
- [results] Urine processing time breakdown: "the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [readme] macOS arm64 warning: [evidence span withheld — anchor status: external_doc]
