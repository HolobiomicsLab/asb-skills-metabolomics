---
name: interactive-umap-plot-generation
description: Use when you have 500-dimensional MS2DeepScore embedding vectors for
  tandem mass spectra and need to visually explore chemical space, especially to check
  whether positive and negative ionization mode spectra of the same compound cluster
  together.
when_to_use_negative:
- Input spectra have not been embedded by MS2DeepScore yet — this skill consumes the
  embedding array, not raw peak lists.
- The embedding quality evaluator flags spectra as unreliable (low information content
  or dissimilar to training data); these poor embeddings will distort the UMAP projection.
- A static, publication-style figure is the only deliverable and interactivity is
  unnecessary — a simple matplotlib scatter of the same 2D coordinates suffices.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MS2DeepScore
  role: Produces the embedding vectors that the UMAP projection visualizes; invoked
    via MS2DeepScore(model).get_embedding_array(cleaned_spectra).
  repo: https://github.com/matchms/ms2deepscore
- name: UMAP
  role: Dimensionality-reduction technique applied to the MS2DeepScore embeddings
    to obtain the 2D chemical-space plot.
- name: matchms
  role: Supplies the preprocessing pipeline (DEFAULT_FILTERS) and the Pipeline object
    whose spectra_queries feed the embedding step before UMAP.
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:0f5410189a987895f4c52326855b51e7142331b180e2edf5eff056c5a44d1dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - UMAP
  - matchms
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# interactive-umap-plot-generation

## Summary

Generates an interactive UMAP scatter plot of MS2DeepScore embeddings, letting users visually explore chemical space by overlaying structural information (e.g. SMILES) on the 2D projection of tandem mass spectra.

## When to use

Use when you have 500-dimensional MS2DeepScore embedding vectors for tandem mass spectra and need to visually explore chemical space, especially to check whether positive and negative ionization mode spectra of the same compound cluster together.

## When NOT to use

- Input spectra have not been embedded by MS2DeepScore yet — this skill consumes the embedding array, not raw peak lists.
- The embedding quality evaluator flags spectra as unreliable (low information content or dissimilar to training data); these poor embeddings will distort the UMAP projection.
- A static, publication-style figure is the only deliverable and interactivity is unnecessary — a simple matplotlib scatter of the same 2D coordinates suffices.

## Inputs

- numpy array of MS2DeepScore embedding vectors (500-dimensional) for each spectrum, produced by MS2DeepScore(model).get_embedding_array()
- preprocessed/cleaned tandem mass spectra in a common format (mgf, msp, mzML, mzXML, json, USI) with metadata such as ionmode and SMILES

## Outputs

- 2D UMAP projection of the embedding vectors (scatter plot coordinates per spectrum)
- interactive HTML UMAP plot (e.g. plotly or bokeh) with per-point hover metadata such as SMILES

## How to apply

First compute the high-dimensional embeddings: pass preprocessed spectra (e.g. in mgf/msp/mzML format, cleaned with the matchms DEFAULT_FILTERS) through the MS2DeepScore model using `MS2DeepScore(model).get_embedding_array(cleaned_spectra)`, which yields a numeric array (the original MS2DeepScore model produces 500-dimensional vectors). Then apply UMAP to reduce these vectors to 2D, using n_neighbors=50 as used in the Rumex sanguineus case study, so that spectra from both ionization modes are projected into one joint embedding plot. Render the result as an interactive HTML figure (e.g. with plotly or bokeh) so hovering individual points can reveal per-spectrum metadata such as the associated SMILES; the README's tutorial shows exactly how to build such an interactive UMAP with overlaid SMILES. The rationale is that the embedding space itself is too high-dimensional to inspect directly, while the UMAP projection directly visualizes predicted cross-ionization-mode chemical similarity: spectra of the same molecule (e.g. Rutin) measured in positive and negative mode should embed close together.

## Related tools

- **MS2DeepScore** (Produces the embedding vectors that the UMAP projection visualizes; invoked via MS2DeepScore(model).get_embedding_array(cleaned_spectra).) — https://github.com/matchms/ms2deepscore
- **UMAP** (Dimensionality-reduction technique applied to the MS2DeepScore embeddings to obtain the 2D chemical-space plot.)
- **matchms** (Supplies the preprocessing pipeline (DEFAULT_FILTERS) and the Pipeline object whose spectra_queries feed the embedding step before UMAP.) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore import MS2DeepScore; ms2ds_model = MS2DeepScore(model); ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- The resulting HTML renders in a browser and shows a scatter plot whose axes span the two UMAP dimensions.
- Each point corresponds one-to-one to an input spectrum (number of points equals the number of embeddings).
- In the Rumex sanguineus case study, spectra of the same molecule in different ionization modes (e.g. Rutin) should embed close together, confirming cross-mode similarity.
- Hovering a point reveals per-spectrum metadata such as SMILES, as shown in the README tutorial figure.
- Re-running UMAP with the same random_state on the same embedding array should reproduce the same projection.

## Limitations

- UMAP projections are stochastic; without a fixed random_state the 2D layout changes between runs.
- Poor-quality spectra (limited fragmentation, chimeric spectra, or spectra dissimilar to the training data) produce unreliable embeddings that will distort the chemical-space visualization.
- The UMAP view only reflects the embedding space; it does not itself validate predicted cross-ionization-mode similarity against measured reference data.

## Evidence

- [intro] UMAP visualization of MS2DeepScore embeddings for exploring chemical space: "We also highlight the possibility of directly visualizing MS2Deepscore embeddings using UMAP22 and provide an interactive plot"
- [readme] README: embeddings can be visualized in chemical space via UMAP: [evidence span withheld — anchor status: external_doc]
- [readme] README: tutorial shows how to create an interactive UMAP with overlaid smiles: [evidence span withheld — anchor status: external_doc]
- [readme] README: MS2DeepScore computes embeddings then chemical similarity scores: [evidence span withheld — anchor status: external_doc]
