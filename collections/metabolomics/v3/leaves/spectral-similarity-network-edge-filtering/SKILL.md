---
name: spectral-similarity-network-edge-filtering
description: Use when a pairwise spectral similarity matrix (e.g. ms2deepscore scores
  from LC–MS/MS .mgf spectra) yields dense, heterogeneous network data and edges must
  be filtered to a sparse local view for exploration.
when_to_use_negative:
- Input is a raw .mgf or MZmine feature list that has not yet been processed through
  the specXplore importing pipeline, which computes the ms2deepscore/modified cosine/spec2vec
  similarities needed for edge filtering.
- Goal is a global, complete molecular network export (e.g. a full GNPS-style network);
  specXplore deliberately avoids visualizing all edges and only draws them on demand
  under user-set filters.
- Dataset contains far more than about 5000 features with liberal settings, where
  the article anticipates dashboard difficulties, or on Windows, which the current
  specXplore version does not support.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: specXplore
  role: Interactive dashboard whose dashboard module applies user-adjustable similarity
    thresholds, top-K neighbor limits, family-size caps, and on-demand edge overlays
    to the t-SNE network view
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the pretrained pairwise spectral similarity scores that serve as
    the quantitative backbone for the edge-thresholding decisions
- name: matchms
  role: Imports the .mgf MS/MS spectra and supplies processing on which similarity
    and fragmentation-overlap values are computed
- name: dash
  role: Framework underlying the interactive dashboard used to change edge filter
    settings on the fly
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:f24b4e63adadc2eba0449bb8215af7dbcd1a177efc61f24d0eb073642613e29a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - dash
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectral-similarity-network-edge-filtering

## Summary

Binarize a pairwise spectral-similarity matrix into a sparse feature network by applying ms2deepscore similarity thresholds, minimum fragmentation overlaps, top-K per-node neighbor limits, and maximal molecular family sizes, then render edges only on demand around a selected node.

## When to use

Use when a pairwise spectral similarity matrix (e.g. ms2deepscore scores from LC–MS/MS .mgf spectra) yields dense, heterogeneous network data and edges must be filtered to a sparse local view for exploration.

## When NOT to use

- Input is a raw .mgf or MZmine feature list that has not yet been processed through the specXplore importing pipeline, which computes the ms2deepscore/modified cosine/spec2vec similarities needed for edge filtering.
- Goal is a global, complete molecular network export (e.g. a full GNPS-style network); specXplore deliberately avoids visualizing all edges and only draws them on demand under user-set filters.
- Dataset contains far more than about 5000 features with liberal settings, where the article anticipates dashboard difficulties, or on Windows, which the current specXplore version does not support.

## Inputs

- ms2deepscore pairwise spectral similarity matrix (from LC–MS/MS spectra in .mgf format)
- Per-pair fragmentation overlap values
- User-adjustable edge filter parameters: similarity threshold, minimum fragmentation overlap, top-K neighbor limit, maximal molecular family size

## Outputs

- Thresholded, sparse edge set (binarized network) rendered as on-demand edge overlays on the t-SNE overview
- Local network/ego-network view around a selected node or feature group, regenerable under new threshold settings

## How to apply

Start from the ms2deepscore pairwise similarity matrix computed in the specXplore importing pipeline, along with the fragmentation overlap information stored for each spectrum pair. Convert continuous similarity scores into edges by imposing a pairwise similarity threshold together with a corresponding minimum fragmentation overlap (the paper cites a 0.7 default for the pairwise threshold), so that only pairs supported by both criteria become candidate edges. Control local density by setting top-K neighbor limits on individual nodes and maximal molecular family sizes, which prevent hubs and large clusters from flooding the map. Because specXplore only visualizes edges on demand using these stringent user-modifiable edge filter settings, select a reference node on the t-SNE overview and regenerate the ego/subnetwork under the new constraints whenever thresholds change, allowing careful expansion of neighborhood size. Rationale: without on-demand stringently filtered edges, dense molecular networking makes the local connectivity structure around a feature of interest unreadable.

## Related tools

- **specXplore** (Interactive dashboard whose dashboard module applies user-adjustable similarity thresholds, top-K neighbor limits, family-size caps, and on-demand edge overlays to the t-SNE network view) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the pretrained pairwise spectral similarity scores that serve as the quantitative backbone for the edge-thresholding decisions)
- **matchms** (Imports the .mgf MS/MS spectra and supplies processing on which similarity and fragmentation-overlap values are computed)
- **dash** (Framework underlying the interactive dashboard used to change edge filter settings on the fly)

## Examples

```
conda activate specxplore_environment && pip install "git+https://github.com/kevinmildau/specXplore.git"  # then run notebooks/demo.ipynb in jupyter-notebook to compute similarities and explore thresholded network views
```

## Evaluation signals

- For a chosen threshold t, every rendered edge must correspond to a similarity pair with score ≥ t and a fragmentation overlap meeting the corresponding minimum; edges failing either condition indicate a filter bug.
- No node may have more than its configured top-K neighbors in the displayed local network; hub nodes exceeding K signal that the neighbor-limit setting was not applied.
- Changing a threshold must visibly regenerate the local view (added or removed edges) within the dashboard session, confirming fast re-filtering rather than a stale overlay.
- The on-demand ego-network around a selected node should stay sparse and readable even on datasets the paper reports as dense (e.g. the urine data set with <4000 features), with no molecular family exceeding the maximal family-size cap.
- K-medoid clustering results should show good agreement with the topological insights from the filtered network, as the paper reports this concordance.

## Limitations

- ms2deepscore scores are only reliable for spectra/metabolites well covered by or related to its training data, so thresholded edges can be misleading for out-of-domain chemistries.
- ms2deepscore predictions on macOS arm64 may silently disagree with other systems due to ms2deepscore issue 199, directly affecting threshold decisions in the importing pipeline.
- The dashboard is expected to scale well to ~5000 features but face difficulties for larger datasets and liberal edge-filter settings; the current version also fails on Windows.
- No tool version numbers are stated, and no automatic guidance exists for choosing similarity thresholds or k-medoid cluster counts; users must tune filters manually.

## Evidence

- [discussion] Default 0.7 pairwise threshold with minimum fragmentation overlaps: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [discussion] Top-K neighbor limits as an edge-filter dimension: "top-K neighbor limits on individual nodes"
- [discussion] Maximal molecular family sizes as an edge-filter dimension: "maximal molecular family sizes"
- [discussion] Edges are drawn only on demand under user-modifiable filters: "avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings."
- [intro] Fast threshold changes regenerate local views and expand neighborhood size: "the ability to quickly change thresholds to regenerate local views under the new constraints, allowing the careful expansion of neighborhood size"
- [readme] README confirms ms2deepscore backbone with t-SNE overview and interactive overlays: [evidence span withheld — anchor status: external_doc]
- [discussion] Scaling caveat for liberal settings on large datasets: "We expect that the dashboard will scale well to 5000 features but face difficulties for larger data sets and liberal settings."
- [readme] macOS arm64 ms2deepscore reliability caveat affecting thresholds: [evidence span withheld — anchor status: external_doc]
