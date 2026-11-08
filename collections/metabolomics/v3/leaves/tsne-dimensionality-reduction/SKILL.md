---
name: tsne-dimensionality-reduction
description: Use when you have high-dimensional MS2DeepScore embeddings (e.g. 200-dim
  vectors from 3601 test-set spectra) and need a 2D visualization to check whether
  spectra group by chemical (super)class.
when_to_use_negative:
- Your goal is to predict Tanimoto structural similarity scores between spectrum pairs
  rather than to visualize embedding structure — use MS2DeepScore scoring directly
  instead
- Your spectra are raw/unprocessed; the embedding step (binning into 10,000 bins over
  10–1000 m/z and base-network inference) must precede t-SNE
- You need a deterministic, quantitatively comparable low-dimension projection for
  downstream numeric analysis rather than exploratory cluster visualization
edam_operation: http://edamontology.org/operation_3893
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore
  role: Produces the 200-dimensional spectrum embeddings (spectrum binning into 10,000
    bins over 10–1000 m/z plus base neural-network inference) that t-SNE projects
    to 2D
  repo: https://github.com/matchms/ms2deepscore
- name: scikit-learn
  role: Provides the t-SNE implementation (metric='cosine', perplexity=100, learning_rate=200,
    1000 iterations) used on the 200-dimensional test-set embeddings
- name: ClassyFire
  role: Supplies the 14 chemical superclass labels used to color the t-SNE scatter
    plot and verify class-wise clustering
- name: matchms
  role: Python library for mass spectra import, handling, and preparation of spectra
    prior to embedding computation
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:2e2855a7867a0a6c6f254c494fe78284dc6af33fd5fba5d5f588826ab59512e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - ClassyFire
  - MS2DeepScore
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3893
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# tsne-dimensionality-reduction

## Summary

Projects 200-dimensional MS2DeepScore spectrum embeddings into 2D with scikit-learn t-SNE (metric='cosine', perplexity=100, learning_rate=200, 1000 iterations) to visually verify that the embeddings carry chemically meaningful information by clustering spectra colored by ClassyFire superclass.

## When to use

Use when you have high-dimensional MS2DeepScore embeddings (e.g. 200-dim vectors from 3601 test-set spectra) and need a 2D visualization to check whether spectra group by chemical (super)class.

## When NOT to use

- Your goal is to predict Tanimoto structural similarity scores between spectrum pairs rather than to visualize embedding structure — use MS2DeepScore scoring directly instead
- Your spectra are raw/unprocessed; the embedding step (binning into 10,000 bins over 10–1000 m/z and base-network inference) must precede t-SNE
- You need a deterministic, quantitatively comparable low-dimension projection for downstream numeric analysis rather than exploratory cluster visualization

## Inputs

- MS/MS spectra for the 3601-spectrum test set (annotated with compound InChIKey)
- 200-dimensional MS2DeepScore embedding vectors (from base-network inference over binned spectra)
- InChIKey-to-ClassyFire superclass mapping (14 chemical superclasses)

## Outputs

- tsne_coordinates.csv: 2D t-SNE coordinates with superclass labels, exactly 3601 rows
- figure_8_tsne.png: scatter plot of 2D coordinates colored by the 14 ClassyFire superclasses

## How to apply

First compute the 200-dimensional embeddings by binning each spectrum into 10,000 equal-width bins over 10–1000 m/z and running inference with the pretrained MS2DeepScore base network (or ms2deepscore's get_embedding_array). Then run t-SNE as implemented in scikit-learn on the 200-dimensional embeddings with metric='cosine', perplexity=100, learning_rate=200, and 1000 iterations to obtain 2D coordinates. Map each spectrum's compound InChIKey to one of the 14 ClassyFire chemical superclasses, save coordinates plus labels as a table (tsne_coordinates.csv, exactly 3601 rows), and render a superclass-colored scatter plot (figure_8_tsne.png). The rationale is that if molecules of the same chemical class cluster together in the t-SNE plot, the embeddings represent chemically meaningful molecular features rather than noise.

## Related tools

- **MS2DeepScore** (Produces the 200-dimensional spectrum embeddings (spectrum binning into 10,000 bins over 10–1000 m/z plus base neural-network inference) that t-SNE projects to 2D) — https://github.com/matchms/ms2deepscore
- **scikit-learn** (Provides the t-SNE implementation (metric='cosine', perplexity=100, learning_rate=200, 1000 iterations) used on the 200-dimensional test-set embeddings)
- **ClassyFire** (Supplies the 14 chemical superclass labels used to color the t-SNE scatter plot and verify class-wise clustering)
- **matchms** (Python library for mass spectra import, handling, and preparation of spectra prior to embedding computation) — https://github.com/matchms/ms2deepscore

## Examples

```
from ms2deepscore import MS2DeepScore; from sklearn.manifold import TSNE; X = MS2DeepScore(model).get_embedding_array(spectra); coords = TSNE(n_components=2, metric='cosine', perplexity=100, learning_rate=200, n_iter=1000).fit_transform(X)
```

## Evaluation signals

- The coordinates table (tsne_coordinates.csv) contains exactly 3601 rows, matching the test-set size
- Every spectrum carries one of the 14 ClassyFire superclass labels
- The colored cluster structure of the scatter plot visually matches Fig. 8 of the paper (same-class molecules cluster together)
- Input embeddings are 200-dimensional vectors obtained via the 10,000-bin (10–1000 m/z) binning plus base-network inference pipeline
- t-SNE was run with the paper's parameters: metric='cosine', perplexity=100, learning_rate=200, 1000 iterations

## Limitations

- Spectral libraries are inherently biased toward certain compound classes, so class-cluster appearance partly reflects this imbalance in the underlying data
- The dataset mixes spectra from different instrumentation types of varying quality and is unbalanced with regard to represented compound classes, which can affect apparent cluster separation
- t-SNE projections are used for qualitative visual confirmation; the article's quantitative claim of chemical meaningfulness rests on the accompanying Tanimoto-prediction results (RMSE 0.13–0.2), not on the plot alone

## Evidence

- [results] t-SNE scikit-learn on 200-dim embeddings: "we ran t-SNE [28] as implemented in scikit-learn [29] on the 200-dimensional embeddings of the test set (3601 spectra)"
- [results] ClassyFire superclass coloring: "colored according to the 14 chemical superclasses provided by ClassyFire [30]"
- [abstract] Chemically meaningful embeddings: "MS2DeepScore can also be used to create chemically meaningful mass spectral embeddings that could be used to cluster large numbers of spectra."
- [results] Embedding creation via binning and base network: "The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network"
- [readme] README: embeddings for dimensionality-reduction visualization: [evidence span withheld — anchor status: external_doc]
