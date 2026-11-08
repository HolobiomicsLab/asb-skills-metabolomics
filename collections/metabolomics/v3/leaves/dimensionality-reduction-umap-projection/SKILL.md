---
name: dimensionality-reduction-umap-projection
description: Use when you have MS2DeepScore embedding vectors (e.g. 500-dim per MS/MS
  spectrum spanning both ionization modes) and need a single 2D chemical-space map,
  like the Rumex sanguineus UMAP.
when_to_use_negative:
- You want similarity scores between spectra, not a 2D map — compute MS2DeepScore
  predictions (similarity matrix) instead of UMAP
- Embeddings are unreliable, e.g. flagged low-quality spectra with limited fragmentation,
  chimeric spectra, or high predicted uncertainty — filter these out before projecting
- The spectra span only one ionization mode and you only need within-mode molecular
  networking (MS2DeepScore scores plus molecular networking suffice)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2DeepScore
  role: Produces the 500-dimensional spectrum embeddings that UMAP reduces to 2D;
    MS2DeepScore 2.0 works across ionization modes so both modes share one embedding
    space
  repo: https://github.com/matchms/MS2DeepScore
- name: UMAP
  role: Dimensionality reduction of MS2DeepScore embeddings to a 2D chemical-space
    projection (n_neighbors=50), optionally rendered as an interactive HTML plot
- name: matchms
  role: DEFAULT_FILTERS pipeline cleans spectra before embedding is computed
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:de170c05c2d4f6888ae2a2d5bf5ad5a27d6f59b22af647e678638e446ff4afe2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore
  - UMAP
  - matchms
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dimensionality-reduction-umap-projection

## Summary

Project 500-dimensional MS2DeepScore 2.0 spectrum embeddings into a 2D UMAP scatter (n_neighbors=50) that mixes positive and negative ionization mode spectra in one chemical-space plot, optionally rendered as an interactive HTML overlay of SMILES.

## When to use

Use when you have MS2DeepScore embedding vectors (e.g. 500-dim per MS/MS spectrum spanning both ionization modes) and need a single 2D chemical-space map, like the Rumex sanguineus UMAP.

## When NOT to use

- You want similarity scores between spectra, not a 2D map — compute MS2DeepScore predictions (similarity matrix) instead of UMAP
- Embeddings are unreliable, e.g. flagged low-quality spectra with limited fragmentation, chimeric spectra, or high predicted uncertainty — filter these out before projecting
- The spectra span only one ionization mode and you only need within-mode molecular networking (MS2DeepScore scores plus molecular networking suffice)

## Inputs

- 500-dimensional MS2DeepScore embedding vectors (one per MS/MS spectrum, from both positive and negative ionization modes)
- Preprocessed MS/MS spectra (mgf, msp, mzml, mzxml, json, usi) passed through matchms DEFAULT_FILTERS
- Per-spectrum metadata for coloring/annotation (e.g. ionization mode, SMILES)

## Outputs

- 2D UMAP coordinate array (n_spectra x 2)
- Static scatter plot colored by ionization mode (reproduction of Figure 3b-style plot)
- Interactive HTML UMAP plot with metadata/SMILES overlay for data exploration

## How to apply

Compute MS2DeepScore embeddings first: load the model (ms2deepscore_model.pt), run the matchms DEFAULT_FILTERS cleaning pipeline over the spectra (mgf/msp/mzml/mzxml/json/usi accepted), and call MS2DeepScore.get_embedding_array(cleaned_spectra) to obtain the 500-dimensional vectors. Feed those vectors into UMAP with n_neighbors=50 as used in the article's Rumex sanguineus case study, keeping positive and negative ionization mode spectra in the same projection rather than splitting by mode. Reduce to 2 components and produce both a static scatter colored by ionization mode and an interactive plot (e.g. plotly/bokeh HTML) with metadata such as SMILES overlaid, since the article explicitly provides 'an interactive plot' alongside UMAP. Rationale: MS2DeepScore 2.0 predicts cross-ionization-mode chemical similarity, so the embedding space is shared across modes and UMAP is the article's recommended direct visualization of that space.

## Related tools

- **MS2DeepScore** (Produces the 500-dimensional spectrum embeddings that UMAP reduces to 2D; MS2DeepScore 2.0 works across ionization modes so both modes share one embedding space) — https://github.com/matchms/MS2DeepScore
- **UMAP** (Dimensionality reduction of MS2DeepScore embeddings to a 2D chemical-space projection (n_neighbors=50), optionally rendered as an interactive HTML plot)
- **matchms** (DEFAULT_FILTERS pipeline cleans spectra before embedding is computed) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore import MS2DeepScore; import umap; emb = MS2DeepScore(model).get_embedding_array(cleaned_spectra); coords = umap.UMAP(n_neighbors=50, n_components=2).fit_transform(emb)
```

## Evaluation signals

- The projection includes spectra from both ionization modes intermixed in one plot, not two separated clouds — e.g. Rutin in positive and negative mode embed closely together
- Input to UMAP is a 500-dimensional embedding array, not the raw spectra or similarity matrix
- n_neighbors=50 was used, matching the article's case-study settings
- Both a static scatter and an interactive HTML file are produced, and the HTML renders the UMAP projection
- Each point maps back to one preprocessed spectrum via the MS2DeepScore embedding pipeline

## Limitations

- Poor MS2DeepScore embeddings (low-quality spectra with limited fragmentation, chimeric spectra, or spectra dissimilar to training data) will distort the projection — use the embedding evaluator / predicted uncertainty to filter spectra first
- UMAP of embeddings is a qualitative exploration aid; it is not the similarity matrix used for molecular networking
- Low-information-content spectra dissimilar to training data yield unreliable embeddings, minimizing such false predictions is recommended before plotting

## Evidence

- [intro] We also highlight the possibility of directly visualizing MS2Deepscore embeddings using UMAP22 and provide an interactive plot: "We also highlight the possibility of directly visualizing MS2Deepscore embeddings using UMAP22 and provide an interactive plot"
- [readme] Interactive UMAP with overlaying smiles: [evidence span withheld — anchor status: not_found]
- [readme] Model works for both ionization modes across modes: [evidence span withheld — anchor status: external_doc]
- [full_text] Spectra of same molecule in different ionization modes embed closely: [evidence span withheld — anchor status: not_found]
- [readme] Compute embeddings via get_embedding_array: "ms2ds_model = MS2DeepScore(model)
ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)"
