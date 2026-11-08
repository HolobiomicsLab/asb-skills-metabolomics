---
name: cross-ionization-mode-spectral-visualization
description: Use when you have processed MS/MS spectra in both ionization modes and
  want a unified chemical-space plot (2D UMAP or interactive HTML) that shows cross-mode
  similarity of the same metabolites.
when_to_use_negative:
- Input spectra are all from a single ionization mode and the goal is standard same-mode
  molecular networking rather than cross-mode comparison
- Input spectra have high predicted uncertainty (low embedding quality) — e.g. low-information
  or chimeric spectra — where the article warns predictions are unreliable and should
  be filtered out first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore
  role: Computes the 500-dimensional embedding vectors for each spectrum that serve
    as input to UMAP; supports both positive and negative ionization modes and cross-mode
    predictions.
  repo: https://github.com/matchms/MS2DeepScore
- name: UMAP
  role: Dimensionality-reduction technique that maps the 500-dimensional MS2DeepScore
    embeddings into a 2D chemical-space plot.
- name: matchms
  role: Provides the DEFAULT_FILTERS preprocessing pipeline (Pipeline / create_workflow)
    used to clean spectra before embedding, and returns cleaned spectra via pipeline.spectra_queries.
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:4c3204fd6057e5b595ccd1459f44fb0d13e769f070832519ae1d0704aa7db5c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore
  - UMAP
  - matchms
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-ionization-mode-spectral-visualization

## Summary

Visualize the combined chemical space of positive and negative ionization mode tandem mass spectra in a single 2D UMAP projection, using 500-dimensional MS2DeepScore 2.0 embeddings so that spectra of the same molecule from different ionization modes cluster together.

## When to use

Use when you have processed MS/MS spectra in both ionization modes and want a unified chemical-space plot (2D UMAP or interactive HTML) that shows cross-mode similarity of the same metabolites.

## When NOT to use

- Input spectra are all from a single ionization mode and the goal is standard same-mode molecular networking rather than cross-mode comparison
- Input spectra have high predicted uncertainty (low embedding quality) — e.g. low-information or chimeric spectra — where the article warns predictions are unreliable and should be filtered out first

## Inputs

- Processed MS/MS spectra from positive and negative ionization modes (MGF, MSP, mzML, MZxml, JSON or USI format)
- Trained MS2DeepScore 2.0 model file (ms2deepscore_model.pt)

## Outputs

- 500-dimensional MS2DeepScore embedding array for all input spectra
- 2D UMAP scatter plot colored by ionization mode
- Interactive HTML UMAP plot for data exploration

## How to apply

Preprocess the spectra (e.g. via the matchms DEFAULT_FILTERS pipeline) for both positive and negative ionization modes, then run them through MS2DeepScore 2.0 to obtain 500-dimensional embeddings using MS2DeepScore(model).get_embedding_array(cleaned_spectra). Reduce these embeddings to 2D with UMAP using n_neighbors=50, and generate a scatter plot colored by ionization mode; the article's Rumex sanguineus case study shows Rutin spectra from both modes embedding closely together. Optionally render an interactive UMAP (e.g. with plotly or bokeh) with overlaying SMILES for exploration, as shown in the ms2deepscore tutorial. Before interpreting the plot, inspect the embedding evaluator output and consider excluding spectra flagged as unreliable or dissimilar to training data.

## Related tools

- **MS2DeepScore** (Computes the 500-dimensional embedding vectors for each spectrum that serve as input to UMAP; supports both positive and negative ionization modes and cross-mode predictions.) — https://github.com/matchms/MS2DeepScore
- **UMAP** (Dimensionality-reduction technique that maps the 500-dimensional MS2DeepScore embeddings into a 2D chemical-space plot.)
- **matchms** (Provides the DEFAULT_FILTERS preprocessing pipeline (Pipeline / create_workflow) used to clean spectra before embedding, and returns cleaned spectra via pipeline.spectra_queries.) — https://github.com/matchms/matchms

## Examples

```
ms2ds_model = MS2DeepScore(model); ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- In the 2D UMAP plot, spectra of the same molecule from different ionization modes (e.g. Rutin in the Rumex sanguineus case study) should embed closely together.
- The embedding array should be 500-dimensional for each input spectrum.
- Positive and negative ionization mode spectra should appear intermixed in the projection rather than forming two fully separate clusters.
- An interactive HTML file should be produced and render the UMAP projection correctly.

## Limitations

- Spectra with low information content or dissimilarity to training data will have unreliable embeddings and can distort the projection; the embedding evaluator can flag these.
- Poor predictions are expected for low-quality spectra with limited fragmentation or chimeric spectra from multiple precursor ions.
- There remains a risk that MS2DeepScore 2.0 is not well-suited for very specific tasks or chemical classes.

## Evidence

- [intro] We also highlight the possibility of directly visualizing MS2Deepscore embeddings using UMAP22 and provide an interactive plot: "We also highlight the possibility of directly visualizing MS2Deepscore embeddings using UMAP22 and provide an interactive plot"
- [abstract] flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions: "flags spectra with low information content or those dissimilar to the training data, thereby minimizing false predictions"
- [results] In the Rumex sanguineus UMAP, spectra of the same molecule from different ionization modes (e.g., Rutin) embed closely together: [evidence span withheld — anchor status: not_found]
- [readme] This intermediate product can also be used to visualize spectra in "chemical space" by using a dimensionality reduction technique, like UMAP.: [evidence span withheld — anchor status: external_doc]
- [readme] The tutorial shows how to use these embeddings to create an interactive UMAP with overlaying smiles.: [evidence span withheld — anchor status: not_found]
