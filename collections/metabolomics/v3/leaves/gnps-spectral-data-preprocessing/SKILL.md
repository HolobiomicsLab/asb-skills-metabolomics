---
name: gnps-spectral-data-preprocessing
description: Use when GNPS MS/MS spectra (e.g. MGF/mzML peak lists) must be transformed
  into fixed 10,000-dimensional vectors before feeding a Siamese network or computing
  spectrum embeddings.
when_to_use_negative:
- Spectra contain fragments above 1000 m/z or below 10 m/z, since peaks outside the
  10–1000 m/z bin range are not captured by this scheme
- The downstream method requires peak-level fragment matching rather than binned vectors
  (e.g. modified cosine comparisons)
- Spectra are already binned/embedded — re-binning would discard the original peak
  structure
edam_operation: http://edamontology.org/operation_2939
edam_topics: []
tools:
- name: ms2deepscore
  role: Provides the binning/embedding pipeline whose binned spectra feed the Siamese
    base network
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Spectrum import, filtering (intensity cutoff, peak cap, square-root transform)
    and handling before binning
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:aa0d411cd022c560bd879d6e9069a67c34b498e7a7a46c4d8cd3bf90f0f78227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gnps-spectral-data-preprocessing

## Summary

Converts GNPS-derived MS/MS spectra into fixed-length numerical vectors by filtering peaks (remove <0.1% of max intensity, cap at 1000 peaks, square-root transform intensities) and binning peaks into 10,000 equal-width bins spanning 10–1000 m/z. These binned vectors are the direct input to the MS2DeepScore Siamese base network before embedding creation.

## When to use

Use when GNPS MS/MS spectra (e.g. MGF/mzML peak lists) must be transformed into fixed 10,000-dimensional vectors before feeding a Siamese network or computing spectrum embeddings.

## When NOT to use

- Spectra contain fragments above 1000 m/z or below 10 m/z, since peaks outside the 10–1000 m/z bin range are not captured by this scheme
- The downstream method requires peak-level fragment matching rather than binned vectors (e.g. modified cosine comparisons)
- Spectra are already binned/embedded — re-binning would discard the original peak structure

## Inputs

- Filtered GNPS-derived MS/MS spectra as peak lists (m/z, intensity pairs), e.g. loaded via matchms from MGF/mzML/msp formats
- matchms Spectrum objects after the intensity/peak-count filtering stage
- Bin definition parameters: 10,000 bins over 10–1000 m/z

## Outputs

- 2D numpy array of shape (n_spectra x 10,000 bins), each row a binned spectrum vector with square-root transformed intensities in occupied bins
- binned_spectra.npy file containing the assembled binned spectra

## How to apply

First filter each spectrum: remove peaks with intensities < 0.1% of the maximum peak intensity, retain at most the 1000 highest-intensity peaks, and square-root transform the intensities (matchms' cleaning pipeline is recommended for this). Next define 10,000 equally-sized bins spanning 10 to 1000 m/z (0.099 m/z per bin) and, for each spectrum, assign each peak to its bin, storing the highest peak intensity contained in that bin and leaving peak-free bins empty. Assemble all binned spectra into a 2D numpy array of shape (n_spectra x 10,000) and save it as binned_spectra.npy. This binning plus the base-network inference step constitutes MS2DeepScore's embedding creation; the score computation itself is comparatively cheap. Decision point: less restrictive (finer) binning was tested and did not significantly decrease the Tanimoto prediction error, so the 10,000-bin scheme is the defensible default.

## Related tools

- **ms2deepscore** (Provides the binning/embedding pipeline whose binned spectra feed the Siamese base network) — https://github.com/matchms/ms2deepscore
- **matchms** (Spectrum import, filtering (intensity cutoff, peak cap, square-root transform) and handling before binning) — https://github.com/matchms/matchms

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS)); pipeline.run('gnps_spectra.mgf')
```

## Evaluation signals

- Each output vector is exactly 10,000 elements wide
- On the training data, 9948 of the 10,000 bins are non-empty, matching the reported bin occupancy
- No peak with intensity < 0.1% of the spectrum's maximum intensity survives filtering, and no spectrum retains more than 1000 peaks
- Occupied bin values correspond to the highest peak intensity in that bin after square-root transformation
- Binned array dimensions equal (number of spectra x 10,000) when assembled from the source spectra list

## Limitations

- Less restrictive (finer) binning did not significantly decrease the overall Tanimoto prediction error in the paper's experiments
- GNPS-derived spectra come from different instrumentation types, varying quality, and unbalanced compound classes, affecting what the binned vectors can represent
- The binned representation is only the first stage of embedding creation; the base-network inference step is the computationally expensive part and scores are not deterministic across retrained models

## Evidence

- [discussion] Binning scheme spans full m/z range: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] Peak filtering thresholds: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [results] Binning precedes base-network inference: "The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network (Fig. 1)"
- [readme] matchms cleaning pipeline recommended: [evidence span withheld — anchor status: external_doc]
