---
name: test-set-spectrum-binning
description: Use when raw MS/MS spectra (peak lists of m/z/intensity pairs) must be
  converted into fixed-length vectors before feeding the MS2DeepScore base network,
  e.g. to compute 200-dim embeddings for t-SNE clustering or similarity scoring.
when_to_use_negative:
- When the input is already a fixed-length feature table or precomputed embedding
  matrix — no binning is needed
- When spectra contain peaks outside the 10–1000 m/z range or require precursor-mass/formula
  inputs (e.g. DeepMass-style approaches) — binning over this fixed range will drop
  out-of-range peaks and MS2DeepScore does not use such metadata
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ms2deepscore
  role: Provides the binning logic and base network that converts binned spectra into
    200-dimensional embeddings
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Imports and filters the spectra (intensity threshold, peak-count limit, sqrt
    transform) before binning
  repo: https://github.com/matchms/matchms
- name: scikit-learn
  role: Consumes the 200-dim embeddings (post-binning inference) for downstream t-SNE
    visualization
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:c23b68b8cf0e3894ca0c779e9946a5ebecd5ed84d44a292fca1153fdec204040
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - ClassyFire
  - MS2DeepScore
  - matchms
  - ms2deepscore
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# test-set-spectrum-binning

## Summary

Bins MS/MS test-set spectra into 10,000 equal-width bins spanning 10–1000 m/z as the first stage of MS2DeepScore embedding creation, so that variable-length peak lists become fixed-size vectors the Siamese base network can consume.

## When to use

Use when raw MS/MS spectra (peak lists of m/z/intensity pairs) must be converted into fixed-length vectors before feeding the MS2DeepScore base network, e.g. to compute 200-dim embeddings for t-SNE clustering or similarity scoring.

## When NOT to use

- When the input is already a fixed-length feature table or precomputed embedding matrix — no binning is needed
- When spectra contain peaks outside the 10–1000 m/z range or require precursor-mass/formula inputs (e.g. DeepMass-style approaches) — binning over this fixed range will drop out-of-range peaks and MS2DeepScore does not use such metadata

## Inputs

- MS/MS spectra of the 3601-spectrum test set (GNPS-derived, annotated with InChIKey) in a matchms-loadable format (e.g. .mgf)
- Pretrained MS2DeepScore base network (ms2deepscore 0.2.0 model)
- matchms Spectrum objects with peak m/z and intensity arrays

## Outputs

- Binned spectrum vectors (10,000 equal-width bins over 10–1000 m/z)
- 200-dimensional MS2DeepScore embeddings per spectrum (input to t-SNE)
- Cached embedding array for reuse in similarity scoring or clustering

## How to apply

For each spectrum, first apply matchms filtering: remove peaks with intensities below 0.1% of the maximum peak intensity, cap at the 1000 highest-intensity peaks, and square-root transform intensities. Then bin the peaks into 10,000 equally-sized bins ranging from 10 to 1000 m/z, aggregating peak intensities into their corresponding bins to yield a fixed-size binned vector. This binning is the first half of the embedding-creation step; the binned vector is then passed through the pretrained MS2DeepScore base network to obtain the 200-dimensional embedding. Binning is computable independently of pair scoring, so cache the binned/embedding vectors once and reuse them for downstream t-SNE or cosine-similarity computations.

## Related tools

- **ms2deepscore** (Provides the binning logic and base network that converts binned spectra into 200-dimensional embeddings) — https://github.com/matchms/ms2deepscore
- **matchms** (Imports and filters the spectra (intensity threshold, peak-count limit, sqrt transform) before binning) — https://github.com/matchms/matchms
- **scikit-learn** (Consumes the 200-dim embeddings (post-binning inference) for downstream t-SNE visualization)

## Examples

```
from ms2deepscore import MS2DeepScore; ms2ds_model = MS2DeepScore(model); ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- Each binned spectrum yields a vector of exactly 10,000 entries covering the 10–1000 m/z window
- Inference on binned test-set spectra produces exactly 3601 embeddings of dimension 200 for the full test set
- Peaks outside 10–1000 m/z and below 0.1% relative intensity must not contribute to any bin
- The embedding array feeds t-SNE (metric='cosine', perplexity=100) whose 2D coordinates table has exactly 3601 rows, one per test-set spectrum

## Limitations

- Scores/embeddings are not deterministic across retrained models; binned inputs may yield different embeddings with a new model
- Spectral libraries are biased toward certain compound classes, which affects what the binned embeddings encode
- Downstream score predictions are not exact compound identifications and are hard to discriminate near-complete matches (Tanimoto 0.8–0.9)

## Evidence

- [methods] Spectrum binning into 10,000 equal-width bins (10–1000 m/z): "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] Pre-binning spectrum filtering rules: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [results] Binning is part of the embedding-creation step: "The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network"
- [readme] README on embeddings from cleaned spectra: [evidence span withheld — anchor status: external_doc]
