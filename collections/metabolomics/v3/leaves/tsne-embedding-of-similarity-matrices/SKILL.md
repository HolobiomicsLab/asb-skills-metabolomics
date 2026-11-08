---
name: tsne-embedding-of-similarity-matrices
description: Use when you have MS/MS feature spectra (.mgf) and need a 2D overview
  layout of pairwise spectral similarities (e.g. ms2deepscore scores) to spot locally
  grouped high-similarity features before detailed network inspection.
when_to_use_negative:
- 'As a standalone clustering/classification result: the paper warns that ''On its
  own, the t-SNE overview figure is difficult to read. Only a limited view of clustering
  trends can be observed'' — use the interactive dashboard or k-medoid clustering
  (which shows good agreement with t-SNE) instead.'
- 'For datasets much larger than ~4000–5000 features: the authors expect the dashboard
  to ''face difficulties for larger data sets and liberal settings'' and scalability
  beyond ~5000 features is untested.'
- 'On macOS arm64 systems with the current ms2deepscore package version: ms2deepscore
  predictions may be silently unreliable (issue 199), which directly affects the similarity
  matrix feeding the embedding.'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Dashboard and importing pipeline whose t-SNE overview embedding of the ms2deepscore
    matrix is the core visualization this skill reproduces
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports .mgf spectral data into Python and performs basic specXplore preprocessing
    before similarity computation
  repo: https://matchms.readthedocs.io/en/latest/
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity scores used as
    the quantitative backbone of the t-SNE embedding
  repo: https://github.com/matchms/ms2deepscore
- name: MS2Query
  role: Supplies the pretrained models used for the machine-learning similarity scores
    feeding the embedding
- name: scikit-learn
  role: Runs the t-SNE dimensionality reduction on the similarity matrix with perplexity
    tuning between 5 and 50
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:f4089029a508d2b995c59c1feadc57fec768827e4866d718556037e8af6fdcf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - scikit-learn
  - specXplore
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# tsne-embedding-of-similarity-matrices

## Summary

Compute a two-dimensional t-SNE embedding of an ms2deepscore pairwise spectral similarity matrix (via scikit-learn) to serve as the overview 'jumping board' for local connectivity exploration of LC-MS/MS feature data in the specXplore dashboard.

## When to use

Use when you have MS/MS feature spectra (.mgf) and need a 2D overview layout of pairwise spectral similarities (e.g. ms2deepscore scores) to spot locally grouped high-similarity features before detailed network inspection.

## When NOT to use

- As a standalone clustering/classification result: the paper warns that 'On its own, the t-SNE overview figure is difficult to read. Only a limited view of clustering trends can be observed' — use the interactive dashboard or k-medoid clustering (which shows good agreement with t-SNE) instead.
- For datasets much larger than ~4000–5000 features: the authors expect the dashboard to 'face difficulties for larger data sets and liberal settings' and scalability beyond ~5000 features is untested.
- On macOS arm64 systems with the current ms2deepscore package version: ms2deepscore predictions may be silently unreliable (issue 199), which directly affects the similarity matrix feeding the embedding.

## Inputs

- .mgf file of MS/MS feature spectra with unique 'feature_id' keys, precursor m/z, and m/z-intensity tuples
- ms2deepscore pairwise similarity matrix (computed in the specXplore importing pipeline or loaded from a prior run)
- pretrained MS2Query model files (positive mode: zenodo.org/records/10527997; negative mode: zenodo.org/records/10528030)

## Outputs

- Two-dimensional t-SNE embedding coordinates, one row per feature (x and y columns, optionally with feature_id), e.g. tsne_embedding.csv
- Perplexity tuning record: the tested perplexity range (5–50) and the selected value
- Input to the specXplore dashboard session for interactive local connectivity exploration

## How to apply

Import the .mgf MS/MS feature spectra into Python with matchms (each entry carrying a 'feature_id' key, precursor m/z, and m/z-intensity tuples), then compute or load the ms2deepscore pairwise similarity matrix using the pretrained MS2Query models (positive or negative mode per data polarity). Run t-SNE from scikit-learn on the similarity matrix, testing perplexity values in the range usually between 5 and 50 and selecting one that balances high-dimensional distance preservation against qualitative grouping of the layout. Write the embedding coordinates (one row per feature with x and y columns, or feature_id/x/y) to a CSV such as tsne_embedding.csv or urine_tsne_embedding.csv. Rationale: the paper's t-SNE overview of the ms2deepscore matrix is deliberately used as the entry point for specXplore's interactive local exploration, not as a standalone clustering result.

## Related tools

- **specXplore** (Dashboard and importing pipeline whose t-SNE overview embedding of the ms2deepscore matrix is the core visualization this skill reproduces) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports .mgf spectral data into Python and performs basic specXplore preprocessing before similarity computation) — https://matchms.readthedocs.io/en/latest/
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity scores used as the quantitative backbone of the t-SNE embedding) — https://github.com/matchms/ms2deepscore
- **MS2Query** (Supplies the pretrained models used for the machine-learning similarity scores feeding the embedding)
- **scikit-learn** (Runs the t-SNE dimensionality reduction on the similarity matrix with perplexity tuning between 5 and 50)

## Examples

```
conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"; jupyter-notebook notebooks/demo.ipynb
```

## Evaluation signals

- Output CSV exists and is well-formed: one coordinate row per wheat/urine feature with finite x/y values (and feature_id field for the urine workflow).
- Expert visual review confirms that high-similarity features occupy nearby positions in the embedding — e.g. wheat data shows a densely connected feature area around the Procyanidin-A2 corresponding feature with characteristic fragment ion overlap.
- Perplexity was swept over the 5–50 range and the selected value is justified by balancing high-dimensional distance preservation against qualitative grouping properties.
- The similarity matrix fed to t-SNE derives from the pretrained MS2Query ms2deepscore model appropriate to the data polarity, computed on a system not affected by ms2deepscore issue 199.
- For the urine dataset (<4000 features), all pairwise spectral similarities completed within roughly the reported ~9 min, indicating the pipeline ran to completion before embedding.

## Limitations

- The t-SNE overview figure is difficult to read on its own; only limited clustering trends are visible without interactivity.
- ms2deepscore can only be expected to work well for spectra and metabolites well covered by or related to its training data; no coverage check is reported.
- On macOS arm64 systems, the current ms2deepscore package may produce similarity predictions inconsistent with other systems (issue 199) — no error or warning is raised, making results silently unreliable.
- specXplore currently works on macOS and Linux but fails on Windows (Windows install additionally requires Microsoft Visual C++ Redistributable for Cython).
- Scalability beyond ~5000 features is untested; larger datasets with liberal settings are expected to cause difficulties.

## Evidence

- [methods] t-SNE on ms2deepscore matrix: [evidence span withheld — anchor status: not_found]
- [methods] perplexity tuning: "test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value."
- [abstract] tsne as jumping board: "provides a two dimensional t-SNE embedding as a jumping board for local connectivity exploration"
- [readme] readme ms2deepscore warning: [evidence span withheld — anchor status: external_doc]
- [results] tsne hard to read alone: "On its own, the t-SNE overview figure is difficult to read. Only a limited view of clustering trends can be observed"
- [results] wheat grouping finding: "K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights"
