---
name: mass-spectral-clustering-visualization
description: Use when you have computed MS2DeepScore embeddings for annotated MS/MS
  spectra (e.g. a 3601-spectrum test set with InChIKeys) and need to verify or display
  that the embeddings group spectra by chemical superclass.
when_to_use_negative:
- When the goal is a similarity-score matrix between spectra pairs rather than an
  embedding-space visualization — use the MS2DeepScore model score computation instead
- When spectra lack InChIKey/structural annotations, since superclass coloring and
  the chemical-clustering check cannot be performed
- When embeddings come from a non-MS2DeepScore pipeline with different dimensionality
  or binning; the 200-dim cosine t-SNE setup is specific to this embedding space
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore
  role: Siamese neural network whose 200-dimensional base-network embeddings are the
    input to the clustering visualization
  repo: https://github.com/matchms/ms2deepscore
- name: scikit-learn
  role: Provides the t-SNE implementation used to project 200-dim embeddings to 2D
- name: ClassyFire
  role: Supplies the 14 chemical superclasses used to color spectra in the t-SNE scatter
    plot
- name: matchms
  role: Library for mass spectra import, handling and comparisons in which MS2DeepScore
    is integrated
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:1ff98ebaf21d0509134d7615ecd8d7b4373f554ff7e208ed56c8193c9f8fca6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - ClassyFire
  - MS2DeepScore
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass-spectral-clustering-visualization

## Summary

Visualize chemically meaningful structure in MS2DeepScore 200-dimensional spectrum embeddings by projecting them to 2D with t-SNE and coloring points by ClassyFire chemical superclass, to check that spectra of the same chemical (super)class cluster together.

## When to use

Use when you have computed MS2DeepScore embeddings for annotated MS/MS spectra (e.g. a 3601-spectrum test set with InChIKeys) and need to verify or display that the embeddings group spectra by chemical superclass.

## When NOT to use

- When the goal is a similarity-score matrix between spectra pairs rather than an embedding-space visualization — use the MS2DeepScore model score computation instead
- When spectra lack InChIKey/structural annotations, since superclass coloring and the chemical-clustering check cannot be performed
- When embeddings come from a non-MS2DeepScore pipeline with different dimensionality or binning; the 200-dim cosine t-SNE setup is specific to this embedding space

## Inputs

- MS/MS spectra with compound InChIKey annotations (e.g. the 3601-spectrum GNPS test set from the MS2DeepScore study)
- Pretrained MS2DeepScore base network model
- ClassyFire superclass assignments per compound (14 superclasses)

## Outputs

- tsne_coordinates.csv — 2D t-SNE coordinates plus superclass labels, one row per spectrum (exactly 3601 rows for the test set)
- figure_8_tsne.png — scatter plot of 2D coordinates colored by chemical superclass

## How to apply

First compute 200-dimensional embeddings via spectrum binning into 10,000 equal-width bins over 10–1000 m/z followed by base-network inference (ms2deepscore get_embedding_array). Then run t-SNE as implemented in scikit-learn with metric='cosine', perplexity=100, learning_rate=200 and 1000 iterations to get 2D coordinates. Map each spectrum's compound InChIKey to one of the 14 ClassyFire chemical superclasses and save the coordinates plus labels (tsne_coordinates.csv, exactly 3601 rows for the test set). Finally render a scatter plot colored by superclass (figure_8_tsne.png); spectra of the same class should visually cluster. Cosine metric is appropriate because embedding cosine similarity is the basis of the MS2DeepScore score computation.

## Related tools

- **MS2DeepScore** (Siamese neural network whose 200-dimensional base-network embeddings are the input to the clustering visualization) — https://github.com/matchms/ms2deepscore
- **scikit-learn** (Provides the t-SNE implementation used to project 200-dim embeddings to 2D)
- **ClassyFire** (Supplies the 14 chemical superclasses used to color spectra in the t-SNE scatter plot)
- **matchms** (Library for mass spectra import, handling and comparisons in which MS2DeepScore is integrated) — https://github.com/matchms/ms2deepscore

## Examples

```
from ms2deepscore import MS2DeepScore; from sklearn.manifold import TSNE; emb = MS2DeepScore(model).get_embedding_array(spectra); coords = TSNE(n_components=2, metric='cosine', perplexity=100, learning_rate=200, n_iter=1000).fit_transform(emb)
```

## Evaluation signals

- tsne_coordinates.csv contains exactly 3601 rows (one per test-set spectrum)
- Every spectrum carries one of the 14 ClassyFire superclass labels
- Molecules of the same chemical class form visually identifiable clusters in the scatter plot, matching Fig. 8 of the paper
- t-SNE was run with metric='cosine', perplexity=100, learning_rate=200 and 1000 iterations on 200-dimensional embeddings

## Limitations

- Spectral libraries are inherently biased toward certain compound classes, which must be considered when interpreting cluster structure
- Predicted scores/embeddings are not deterministic and may change when a new model is trained (different parameters or training data)
- The current pretrained model differs from the 2021 paper version: the README's model was trained on >500,000 combined GNPS/Mona/MassBank/MSnLib spectra and works across positive and negative ionization modes, so reproduced plots may not exactly match the paper

## Evidence

- [results] t-SNE on 200-dim embeddings: "we ran t-SNE [28] as implemented in scikit-learn [29] on the 200-dimensional embeddings of the test set (3601 spectra)"
- [results] superclass coloring: "colored according to the 14 chemical superclasses provided by ClassyFire [30]"
- [discussion] embedding creation steps: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [abstract] chemically meaningful embeddings: "MS2DeepScore can also be used to create chemically meaningful mass spectral embeddings that could be used to cluster large numbers of spectra."
- [readme] README embedding visualization: [evidence span withheld — anchor status: external_doc]
