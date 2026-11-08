---
name: chemical-superclass-annotation
description: Use when you have MS/MS spectra with InChIKey annotations and 2D embeddings
  (e.g., t-SNE or UMAP of MS2DeepScore vectors) and need to check whether the projection
  groups spectra by chemical (super)class.
when_to_use_negative:
- Spectra lack InChIKey (or equivalent structure) annotations, so no ClassyFire superclass
  can be assigned
- You need per-compound classification beyond the 14 ClassyFire superclasses (e.g.,
  fine-grained chemical classes) — this skill only validates coarse superclass-level
  clustering
- The goal is predicting Tanimoto similarity scores between spectrum pairs rather
  than validating embedding chemical meaningfulness via class-colored visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0121
tools:
- name: ClassyFire
  role: Provides the 14 chemical superclasses that InChIKeys are mapped to for coloring
    the projection
- name: MS2DeepScore
  role: Produces the 200-dimensional spectrum embeddings that are projected and superclass-colored
  repo: https://github.com/matchms/ms2deepscore
- name: scikit-learn
  role: Runs t-SNE on the 200-dimensional embeddings to obtain 2D coordinates for
    the colored scatter plot
- name: matchms
  role: Mass spectra import, handling and preprocessing (e.g., cleaning pipeline)
    prior to embedding
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:af9196b5122cf15928bb49f860bef5c61a979fd7d8b1ac026e6e74308ccfd809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - ClassyFire
  - MS2DeepScore
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chemical-superclass-annotation

## Summary

Annotates mass spectra (via their compounds' InChIKeys) with one of 14 ClassyFire chemical superclasses and overlays these labels on 2D dimensionality-reduction plots (t-SNE of 200-dim MS2DeepScore embeddings) to verify that embeddings cluster spectra by chemistry.

## When to use

Use when you have MS/MS spectra with InChIKey annotations and 2D embeddings (e.g., t-SNE or UMAP of MS2DeepScore vectors) and need to check whether the projection groups spectra by chemical (super)class.

## When NOT to use

- Spectra lack InChIKey (or equivalent structure) annotations, so no ClassyFire superclass can be assigned
- You need per-compound classification beyond the 14 ClassyFire superclasses (e.g., fine-grained chemical classes) — this skill only validates coarse superclass-level clustering
- The goal is predicting Tanimoto similarity scores between spectrum pairs rather than validating embedding chemical meaningfulness via class-colored visualization

## Inputs

- 3601 test-set MS/MS spectra with compound InChIKey annotations
- 200-dimensional MS2DeepScore embeddings (from spectrum binning into 10,000 equal-width 10–1000 m/z bins plus base-network inference)
- ClassyFire superclass mapping (14 chemical superclasses) for InChIKeys

## Outputs

- tsne_coordinates.csv: 2D t-SNE coordinates with one of 14 ClassyFire superclass labels per spectrum (3601 rows)
- figure_8_tsne.png: scatter plot of 2D coordinates colored by superclass

## How to apply

Map each spectrum's compound InChIKey to one of the 14 ClassyFire chemical superclasses, then color each point of the 2D projection (e.g., t-SNE with metric='cosine', perplexity=100, learning_rate=200, 1000 iterations, run in scikit-learn on the 200-dimensional MS2DeepScore embeddings of the 3601-spectrum test set) by its superclass label. Persist the 2D coordinates together with superclass labels as a table (tsne_coordinates.csv, exactly 3601 rows in the paper's setup) and render a colored scatter plot (figure_8_tsne.png). The rationale is that if molecules of the same chemical class cluster together in the plot, the underlying embeddings encode chemically meaningful molecular features. Each spectrum must carry exactly one of the 14 superclass labels before plotting.

## Related tools

- **ClassyFire** (Provides the 14 chemical superclasses that InChIKeys are mapped to for coloring the projection)
- **MS2DeepScore** (Produces the 200-dimensional spectrum embeddings that are projected and superclass-colored) — https://github.com/matchms/ms2deepscore
- **scikit-learn** (Runs t-SNE on the 200-dimensional embeddings to obtain 2D coordinates for the colored scatter plot)
- **matchms** (Mass spectra import, handling and preprocessing (e.g., cleaning pipeline) prior to embedding) — https://github.com/matchms/ms2deepscore

## Examples

```
ms2ds_model = MS2DeepScore(model); ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- The coordinates table contains exactly 3601 rows (one per test-set spectrum)
- Every spectrum carries exactly one of the 14 ClassyFire superclass labels
- Colored cluster structure of the scatter plot visually matches Fig. 8 of the paper (same-class molecules clustering together)
- Embeddings used are 200-dimensional vectors obtained via base-network inference, not raw binned spectra

## Limitations

- Spectral libraries are biased toward certain compound classes, so superclass coverage and clustering appearance depend on the dataset
- t-SNE output is not deterministic; the specific 2D layout depends on parameters (perplexity=100, learning_rate=200, 1000 iterations) and random seed
- Clustering by superclass only shows coarse chemical meaningfulness — it does not enable exact compound identification

## Evidence

- [results] colored according to the 14 chemical superclasses provided by ClassyFire [30]: "colored according to the 14 chemical superclasses provided by ClassyFire [30]"
- [results] t-SNE run in scikit-learn on test-set embeddings: "we ran t-SNE [28] as implemented in scikit-learn [29] on the 200-dimensional embeddings of the test set (3601 spectra)"
- [abstract] Chemically meaningful embeddings for clustering: "MS2DeepScore can also be used to create chemically meaningful mass spectral embeddings that could be used to cluster large numbers of spectra."
- [readme] README: embeddings for chemical-space visualization: [evidence span withheld — anchor status: external_doc]
