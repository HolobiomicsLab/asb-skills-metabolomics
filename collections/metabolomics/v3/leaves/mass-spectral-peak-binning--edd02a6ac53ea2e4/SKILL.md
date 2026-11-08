---
name: mass-spectral-peak-binning
description: Use when filtered MS/MS peak lists must be turned into fixed-width numeric
  vectors (10,000 bins over 10–1000 m/z) to feed the MS2DeepScore base network or
  to build 200-dimensional spectrum embeddings.
when_to_use_negative:
- Spectra have already been binned or embedded (e.g. 200-dimensional MS2DeepScore
  embeddings already computed) — re-binning would discard information
- The goal is a classical similarity measure such as (modified) cosine or Spec2Vec,
  which operate on peaks or learned vectors rather than the binned network input
- Spectra contain peaks outside the 10–1000 m/z range that are essential to the analysis,
  since the binning scheme discards anything outside these bounds and collapses sub-bin
  resolution
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore
  role: Target model whose input preparation this binning serves; embedding creation
    includes the mass spectrum binning plus inference with the base neural network
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library for mass spectra import, handling, and the cleaning/filtering
    pipeline applied before binning
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:7a8b52b2a93605127b73042ba4698bd4cfebff390b0c2af8d8709dc6c3e23cb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mass-spectral-peak-binning

## Summary

Converts filtered MS/MS spectra into fixed-length numeric vectors by assigning each peak to one of 10,000 equally-sized bins spanning 10–1000 m/z, keeping the highest peak intensity per bin. This binning is the first stage of input preparation for the MS2DeepScore Siamese base network and is part of the embedding creation step.

## When to use

Use when filtered MS/MS peak lists must be turned into fixed-width numeric vectors (10,000 bins over 10–1000 m/z) to feed the MS2DeepScore base network or to build 200-dimensional spectrum embeddings.

## When NOT to use

- Spectra have already been binned or embedded (e.g. 200-dimensional MS2DeepScore embeddings already computed) — re-binning would discard information
- The goal is a classical similarity measure such as (modified) cosine or Spec2Vec, which operate on peaks or learned vectors rather than the binned network input
- Spectra contain peaks outside the 10–1000 m/z range that are essential to the analysis, since the binning scheme discards anything outside these bounds and collapses sub-bin resolution

## Inputs

- Filtered MS/MS spectra (peaks <0.1% of max intensity removed, max 1000 highest-intensity peaks kept, square-root transformed intensities), e.g. as peak lists or matchms Spectrum objects
- Binning scheme parameters: 10,000 equally-sized bins over 10–1000 m/z

## Outputs

- A 2D numpy array of binned spectra with shape (n_spectra, 10000), e.g. saved as binned_spectra.npy
- Per-spectrum 10,000-dimensional vectors suitable for the MS2DeepScore base network / embedding creation

## How to apply

First ensure spectra are filtered as in the MS2DeepScore pipeline: peaks below 0.1% of the maximum peak intensity removed, at most the 1000 highest-intensity peaks retained, and intensities square-root transformed. Load the filtered spectra into numpy arrays, then define 10,000 equally-sized bins ranging from 10 to 1000 m/z. For each spectrum, assign each peak to its bin and set that bin's value to the highest peak intensity it contains (do not sum intensities), leaving bins without peaks empty. Assemble all binned spectra into a 2D numpy array of shape (n_spectra, 10000) and save it (e.g. binned_spectra.npy); this binned representation feeds the base neural network whose inference yields the 200-dimensional embeddings used for cosine-similarity score prediction.

## Related tools

- **MS2DeepScore** (Target model whose input preparation this binning serves; embedding creation includes the mass spectrum binning plus inference with the base neural network) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library for mass spectra import, handling, and the cleaning/filtering pipeline applied before binning) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore import MS2DeepScore; ms2ds_model = MS2DeepScore(model); ms2ds_embeddings = ms2ds_model.get_embedding_array(cleaned_spectra)
```

## Evaluation signals

- On the training data, exactly 9948 of the 10,000 bins are non-empty, matching the reported bin occupancy
- Each output vector is exactly 10,000 elements wide, with one row per input spectrum
- Each bin's value equals the highest (not summed) square-root-transformed peak intensity among peaks assigned to it; bins without peaks remain zero/empty
- Downstream: the binned vectors fed through the base network yield 200-dimensional embeddings whose cosine similarities produce Tanimoto predictions with RMSE between 0.13 and 0.2 on the test set

## Limitations

- Binning collapses peaks within each bin and discards peaks outside 10–1000 m/z; the paper reports that less restrictive (finer) binning could not significantly decrease the overall Tanimoto prediction error, suggesting limited gains from changing the scheme
- The binning scheme is tied to the MS2DeepScore input pipeline (10–1000 m/z, 10,000 bins); other models or spectral libraries may require different preprocessing
- MS2DeepScore scores built on these binned inputs are not deterministic and may change when the model is retrained with different parameters or training data

## Evidence

- [discussion] Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z.: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [results] The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network (Fig. 1).: "The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network (Fig. 1)"
- [discussion] Filtering thresholds applied before binning (discussion).: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [readme] Embeddings as intermediate product representing each spectrum (README).: [evidence span withheld — anchor status: external_doc]
