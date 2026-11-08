---
name: feature-prioritization-interpretation
description: Use when a globaltest-based prioritization on msFeaST spectral-similarity
  clusters has flagged differential features and you must assess whether their MS2-similarity
  connectivity supports the statistical call.
when_to_use_negative:
- Input is raw LC-MS/MS spectra rather than an already-preprocessed quantification
  feature table with an accompanying similarity matrix — preprocessing belongs to
  the msFeaST preprocessing notebook, not this interpretation step.
- The analysis goal is de novo feature grouping rather than interpreting statistically
  prioritized features from an existing globaltest run on spectral-similarity clusters.
- The dashboard bundle and preprocessed .json data are unavailable and only raw example
  files exist — without the pipeline output this skill cannot be executed as described.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: msFeaST
  role: Provides the interactive integration and visualization of semi-quantitative
    experimental information with mass-spectral structural similarity information
    used to interpret prioritized features; the dashboard bundle and pipeline notebooks
    supply the similarity matrix and network view.
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Performs the feature group-wise statistical testing of k-medoids spectral-similarity
    clusters whose top-prioritized feature is the subject of the neighborhood interpretation.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:d170e3ec7616b0bcb62a5b8cee845c79475af3f9dfb10c9551e4b0e1ccd21448
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# feature-prioritization-interpretation

## Summary

Interpret and validate statistically prioritized LC-MS/MS features from the msFeaST workflow by examining their spectral-similarity neighborhoods (e.g. top-30 nearest-neighbor edges) in the interactive dashboard, verifying that a globally tested feature connects beyond its k-medoids cluster.

## When to use

Use when a globaltest-based prioritization on msFeaST spectral-similarity clusters has flagged differential features and you must assess whether their MS2-similarity connectivity supports the statistical call.

## When NOT to use

- Input is raw LC-MS/MS spectra rather than an already-preprocessed quantification feature table with an accompanying similarity matrix — preprocessing belongs to the msFeaST preprocessing notebook, not this interpretation step.
- The analysis goal is de novo feature grouping rather than interpreting statistically prioritized features from an existing globaltest run on spectral-similarity clusters.
- The dashboard bundle and preprocessed .json data are unavailable and only raw example files exist — without the pipeline output this skill cannot be executed as described.

## Inputs

- msFeaST quantification feature table (preprocessed LC-MS/MS untargeted metabolomics data)
- precomputed pairwise spectral similarity matrix (e.g. MS2Deepscore-based)
- metadata table defining the contrast (e.g. 0% vs 80% olive mill solid waste substrate)
- dashboard_data.json produced by the msfeast_pipeline notebook

## Outputs

- edge list of the prioritized feature's top-30 nearest neighbors with pairwise similarity scores (fig2b_top30_edges.csv)
- visual assessment of network connectivity of the differential feature beyond its k-medoids cluster
- interpretation of whether prioritized features show strong spectral-similarity connectivity

## How to apply

Load the msFeaST illustrative-example or user-produced feature table and the precomputed pairwise spectral similarity matrix produced by the msFeaST pipeline (e.g. notebooks/data/omsw_pleurotus_ms2deepscore/dashboard_data.json). Identify the top-prioritized feature from the globaltest-based group-wise testing of the k-medoids spectral-similarity clusters (e.g. the Pleurotus eryngii 0% vs 80% olive mill solid waste contrast highlighted in Fig. 2B). Retrieve the similarity values between that feature and all other features, rank them descending, and select the top-30 nearest neighbors as (feature_A, feature_B, similarity) edge records. Overlay these edges onto the feature network in the msFeaST_Dashboard_bundle.html to check that connectivity extends beyond the feature's own small k-medoid cluster (cf. Supplementary Fig. S1), using the pairwise similarity scores to judge connection strength.

## Related tools

- **msFeaST** (Provides the interactive integration and visualization of semi-quantitative experimental information with mass-spectral structural similarity information used to interpret prioritized features; the dashboard bundle and pipeline notebooks supply the similarity matrix and network view.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Performs the feature group-wise statistical testing of k-medoids spectral-similarity clusters whose top-prioritized feature is the subject of the neighborhood interpretation.)

## Examples

```
conda activate msfeast_environment && jupyter-notebook
```

## Evaluation signals

- The exported edge list contains exactly 30 edges, one row per neighbor, each carrying a pairwise similarity score.
- Edges are ranked by descending pairwise spectral similarity between the differential feature and its neighbors.
- Overlaying the edges shows the differential feature's connectivity extends beyond its own small k-medoids cluster, as reported in Section 3 / Supplementary Fig. S1.
- Edge similarities are consistent with values in the source pairwise spectral similarity matrix (no rescaling or re-ranking).
- The dashboard loads the pipeline-produced dashboard_data.json without errors and the dataview tab shows the loaded data.

## Limitations

- msFeaST preprocessing and pipeline were tested on macOS and should work identically on Linux; Windows support was untested and in progress at publication time.
- The R dependency stack is version-pinned (R 4.3.3, globaltest 5.56.0 via Bioconductor 3.18, Matrix 1.6-5, survival 3.5-8, etc.) and conda R package installation is unreliable, so rscript path caching issues can break installation.
- Windows users must run commands from the ANACONDA PROMPT; terminal environments (e.g. vscode) that pre-activate a different conda environment can cause R path misalignment.

## Evidence

- [abstract] msFeaST presented as a feature-set testing and visualization workflow: "Here, we present msFeaST (mass spectral Feature Set Testing), a feature-set testing and visualization workflow for LC-MS/MS untargeted metabolomics data."
- [abstract] globaltest testing on spectral clusters: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
- [abstract] interactive integration of experimental and spectral similarity information: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information"
- [abstract] k-medoids clustering as structural grouping proxy: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] dashboard bundle and example data location: [evidence span withheld — anchor status: external_doc]
- [readme] pinned R versions including globaltest 5.56.0: [evidence span withheld — anchor status: external_doc]
