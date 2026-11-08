---
name: perplexity-parameter-tuning
description: Use when computing the specXplore t-SNE overview embedding from an ms2deepscore
  pairwise similarity matrix (e.g. wheat <1000 or urine <4000 features) and a perplexity
  value must be chosen before exporting tsne_embedding.csv.
when_to_use_negative:
- When the goal is an interactive local-neighborhood exploration (thresholds, top-K
  neighbor limits, fragmentation overlaps) — the t-SNE overview is only a starting
  point and is difficult to read on its own
- When a supervised or reference-based structural annotation is needed (e.g. MS2Query
  library search), since perplexity tuning addresses only the unsupervised 2D layout
  of pairwise similarities
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: specXplore
  role: Pipeline and dashboard whose t-SNE overview embedding of the ms2deepscore
    similarity matrix requires the tuned perplexity
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity matrix that is
    the input to the t-SNE embedding being tuned
- name: MS2Query
  role: Supplies the pretrained ms2deepscore model used to compute the similarity
    scores feeding the embedding
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:eb409973c5ace189c5380c1e954463da00c066f3dadce74af702b1ffacf484ef
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# perplexity-parameter-tuning

## Summary

Selects an appropriate t-SNE perplexity value (typically 5–50) when embedding an ms2deepscore pairwise similarity matrix, balancing preservation of high-dimensional distances against qualitative grouping properties in the two-dimensional layout used by specXplore.

## When to use

Use when computing the specXplore t-SNE overview embedding from an ms2deepscore pairwise similarity matrix (e.g. wheat <1000 or urine <4000 features) and a perplexity value must be chosen before exporting tsne_embedding.csv.

## When NOT to use

- When the goal is an interactive local-neighborhood exploration (thresholds, top-K neighbor limits, fragmentation overlaps) — the t-SNE overview is only a starting point and is difficult to read on its own
- When a supervised or reference-based structural annotation is needed (e.g. MS2Query library search), since perplexity tuning addresses only the unsupervised 2D layout of pairwise similarities

## Inputs

- ms2deepscore pairwise similarity matrix (square, one row/column per feature) computed by the specXplore importing pipeline
- MGF-format MS/MS feature spectra with feature_id metadata keys (wheat or urine LC-MS/MS feature lists)

## Outputs

- Two-dimensional t-SNE embedding coordinates (one row per feature, x and y columns, e.g. tsne_embedding.csv or urine_tsne_embedding.csv)
- Selected perplexity value with rationale for the chosen setting

## How to apply

Run scikit-learn's t-SNE on the ms2deepscore pairwise similarity matrix, which serves as the primary similarity score in the specXplore pipeline. Test a range of perplexity values (usually between 5 and 50) rather than accepting the default, since the perplexity controls the local neighborhood size that governs both high-dimensional distance preservation and qualitative grouping. Judge candidate values by whether the resulting 2D layout keeps high-ms2deepscore-similarity features near each other (e.g. the densely connected area around the Procyanidin-A2 feature in the wheat data) while retaining limited global clustering trends. Once selected, write one row per feature with x and y coordinates to the embedding CSV; k-medoid clustering is expected to show good agreement with the chosen projection, which provides a cross-check.

## Related tools

- **specXplore** (Pipeline and dashboard whose t-SNE overview embedding of the ms2deepscore similarity matrix requires the tuned perplexity) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity matrix that is the input to the t-SNE embedding being tuned)
- **MS2Query** (Supplies the pretrained ms2deepscore model used to compute the similarity scores feeding the embedding)

## Examples

```
from sklearn.manifold import TSNE; coords = TSNE(n_components=2, perplexity=30, metric='precomputed').fit_transform(ms2deepscore_matrix)
```

## Evaluation signals

- Embedding CSV exists, is well-formed, and has one coordinate row per feature with finite x/y values
- Features with high ms2deepscore similarity occupy nearby positions in the embedding (e.g. fragment-ion-overlapping features group around the Procyanidin-A2 reference standard in the wheat data)
- Perplexity was selected from a tested range between 5 and 50 rather than left at the library default
- K-medoid clustering of the similarity matrix shows good agreement with the t-SNE projection and topological insights
- Expert visual review confirms grouping of high-similarity features is consistent with the paper's illustrative wheat example

## Limitations

- The static t-SNE overview figure is difficult to read on its own; only limited clustering trends are visible without interactivity
- macos arm64 users should note ms2deepscore issue 199: current ms2deepscore package versions can produce similarity predictions not in accordance with results on other systems, silently corrupting the similarity matrix that the embedding is built on
- ms2deepscore can only be expected to work well for spectra and metabolites well covered by or related to its training data, so the embedding quality degrades outside that domain

## Evidence

- [methods] test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value.: "test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value."
- [results] On its own, the t-SNE overview figure is difficult to read. Only a limited view of clustering trends can be observed: "On its own, the t-SNE overview figure is difficult to read. Only a limited view of clustering trends can be observed"
- [results] K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights: "K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights"
- [readme] The current ms2deepscore package version may lead to ms2deepscore similarity predictions that are not in accordance with results on other systems: [evidence span withheld — anchor status: external_doc]
