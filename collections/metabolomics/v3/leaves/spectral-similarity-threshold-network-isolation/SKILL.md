---
name: spectral-similarity-threshold-network-isolation
description: Use when a specXplore session object (ms2deepscore similarity matrix
  + t-SNE embedding) exists for an LC–MS/MS dataset and you need the densely connected
  subnetwork around a feature (e.g. Procyanidin-A2) isolated by similarity thresholds
  for fragmentation-overlap analysis.
when_to_use_negative:
- Dataset exceeds roughly 5000 features and liberal threshold settings are needed
  — the dashboard is only expected to scale well to 5000 features and may face difficulties
  beyond
- Running in Windows — the current specXplore version 'works on Macos and Linux but
  fails in windows'
- Spectra/metabolites are poorly covered by or unrelated to ms2deepscore's training
  data, so the similarity backbone is not trustworthy
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Provides the threshold/connectivity-limitation settings and partial network
    drawings used to isolate and explore the local neighborhood around a target feature
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Supplies the precomputed pairwise spectral similarity matrix whose thresholding
    defines the network edges
- name: matchms
  role: Imports the .mgf MS/MS spectra and provides Spectrum objects for retrieving
    (m/z, intensity) peak tuples of the isolated region
  repo: https://matchms.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:a084a4f6f354f9c3e381237ef7658866fdc1bcf06637b05100187f8dd75bb1c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# spectral-similarity-threshold-network-isolation

## Summary

Isolates the local network neighborhood of a target MS/MS feature by thresholding a precomputed ms2deepscore pairwise spectral similarity matrix (with fragmentation-overlap and top-K connectivity constraints) so that fragmentation-pattern overlaps and metadata around dense feature regions can be inspected for structural hypothesis generation.

## When to use

Use when a specXplore session object (ms2deepscore similarity matrix + t-SNE embedding) exists for an LC–MS/MS dataset and you need the densely connected subnetwork around a feature (e.g. Procyanidin-A2) isolated by similarity thresholds for fragmentation-overlap analysis.

## When NOT to use

- Dataset exceeds roughly 5000 features and liberal threshold settings are needed — the dashboard is only expected to scale well to 5000 features and may face difficulties beyond
- Running in Windows — the current specXplore version 'works on Macos and Linux but fails in windows'
- Spectra/metabolites are poorly covered by or unrelated to ms2deepscore's training data, so the similarity backbone is not trustworthy

## Inputs

- Precomputed ms2deepscore pairwise similarity matrix from the specXplore importing pipeline
- t-SNE embedding of the ms2deepscore similarity matrix (specXplore session data object)
- .mgf file containing MS/MS peak tuples with 'feature_id' metadata keys
- Feature identifier of the target node of interest (e.g. the Procyanidin-A2 corresponding feature)

## Outputs

- Isolated local network neighborhood (partial network drawing / node-edge set) around the target feature
- Pairwise fragment-ion overlap fractions among all spectra in the isolated region (fragmap-style)
- Summary statistics of the ms2deepscore similarity distribution within the region (e.g. procyanidin_a2_region_overlap_stats.csv)

## How to apply

Load the precomputed ms2deepscore pairwise similarity matrix and the t-SNE embedding produced by the specXplore importing pipeline (which ingests .mgf files with a 'feature_id' key via matchms), and locate the target feature node on the t-SNE overview. Threshold the similarity matrix to retain edges meeting the tool's default pairwise settings (minimum fragmentation overlap of 0.7), optionally applying top-K neighbor limits on individual nodes and maximal molecular family sizes to control density. Extract the resulting local neighborhood as a partial network drawing, then retrieve the (m/z, intensity) peak tuples for all spectra in the region from the .mgf data using matchms. Compute pairwise fragment-ion overlap fractions by comparing binned and sorted m/z fragments, in the manner of the specXplore fragmap construction, and summarize the ms2deepscore similarity distribution (bounded in [0, 1]) within the region. Iterate by quickly changing thresholds to regenerate local views under new constraints, allowing careful expansion of neighborhood size.

## Related tools

- **specXplore** (Provides the threshold/connectivity-limitation settings and partial network drawings used to isolate and explore the local neighborhood around a target feature) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Supplies the precomputed pairwise spectral similarity matrix whose thresholding defines the network edges)
- **matchms** (Imports the .mgf MS/MS spectra and provides Spectrum objects for retrieving (m/z, intensity) peak tuples of the isolated region) — https://matchms.readthedocs.io/en/latest/

## Examples

```
conda create --name specxplore_environment python=3.8; conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- The isolated region contains a nonzero feature count, and fragment-overlap and similarity-distribution fields are populated in the output table
- ms2deepscore similarity values within the region remain bounded in [0, 1]
- Computed fragment-ion overlap statistics reproduce the characteristic fragmentation overlap the paper reports for the target feature group (e.g. the Procyanidin-A2 feature region)
- Varying the threshold (e.g. from the default minimum fragmentation overlap of 0.7) regenerates sensible, expandable local views rather than disconnected or saturated graphs
- Re-running the thresholding on the same similarity matrix yields an identical node/edge neighborhood (deterministic threshold application)

## Limitations

- ms2deepscore applicability is limited to spectra/metabolites covered by or related to its training data, with no coverage check reported
- On macos arm64 systems, ms2deepscore issue 199 can produce unreliable similarity predictions without any error or warning, directly affecting the importing pipeline's default similarity scores
- The pairwise similarity thresholds interact with minimum fragmentation overlaps (default 0.7), top-K neighbor limits, and maximal molecular family sizes; overly liberal settings on large datasets risk dashboard difficulties beyond ~5000 features
- No online hosted version of specXplore exists; local installation is required

## Evidence

- [abstract] SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone, while allowing fast changes of threshold and connectivity limitation settings: "SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone, while allowing fast changes of threshold and connectivity limitation settings"
- [methods] based on a t-SNE embedding of the ms2deepscore pairwise similarity matrix serving as the primary similarity score.: "based on a t-SNE embedding of the ms2deepscore pairwise similarity matrix serving as the primary similarity score."
- [discussion] the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [readme] SpecXplore includes network views, similarity heatmaps, and fragmentation overview maps.: [evidence span withheld — anchor status: external_doc]
