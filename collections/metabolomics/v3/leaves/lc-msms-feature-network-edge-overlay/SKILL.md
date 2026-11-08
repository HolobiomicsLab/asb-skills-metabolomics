---
name: lc-msms-feature-network-edge-overlay
description: Use when a globaltest-prioritized LC-MS/MS differential feature must
  be inspected in its spectral-similarity network context — e.g. to check whether
  its strongest neighbors (top-30 edges) reach beyond its small k-medoid cluster.
when_to_use_negative:
- No pairwise spectral similarity matrix is available for the LC-MS/MS features (edges
  cannot carry pairwise similarity information).
- The goal is feature-group statistical prioritization itself (globaltest testing
  of spectral-similarity clusters), not post-hoc network inspection of an already-prioritized
  feature.
- Input is raw LC-MS/MS spectra that have not yet been preprocessed into a quantification/feature
  table by the msFeaST pipeline notebooks.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Supplies the feature table, precomputed spectral-similarity matrix, k-medoids
    feature grouping, and the interactive dashboard (msFeaST_Dashboard_bundle.html)
    on which the top-N neighbor edges are overlaid.
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Feature group-wise statistical testing used to identify the top-prioritized
    differential feature whose top-30 neighbor edges are extracted.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:f278b7217d855da341ad5c648e50fdb51d62c761aae85f38b29c9a1494508375
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lc-msms-feature-network-edge-overlay

## Summary

Builds a top-N neighbor edge list (feature_A, feature_B, spectral-similarity) for a prioritized LC-MS/MS differential feature and overlays it on the msFeaST interactive feature network to assess whether connectivity extends beyond the feature's own k-medoids spectral-similarity cluster.

## When to use

Use when a globaltest-prioritized LC-MS/MS differential feature must be inspected in its spectral-similarity network context — e.g. to check whether its strongest neighbors (top-30 edges) reach beyond its small k-medoid cluster.

## When NOT to use

- No pairwise spectral similarity matrix is available for the LC-MS/MS features (edges cannot carry pairwise similarity information).
- The goal is feature-group statistical prioritization itself (globaltest testing of spectral-similarity clusters), not post-hoc network inspection of an already-prioritized feature.
- Input is raw LC-MS/MS spectra that have not yet been preprocessed into a quantification/feature table by the msFeaST pipeline notebooks.

## Inputs

- msFeaST feature table (quantification table) for the illustrative Pleurotus eryngii 0% vs 80% olive mill solid waste contrast
- Precomputed pairwise spectral similarity matrix (MS2/structural similarity scores) from the msFeaST repository example data
- Identifier of the top-prioritized differential feature from globaltest-based feature-group testing (the Fig. 2B feature)
- Fig. 2B feature network / dashboard data (notebooks/data/omsw_pleurotus_ms2deepscore/dashboard_data.json) for overlay

## Outputs

- fig2b_top30_edges.csv: edge list with exactly 30 rows of (feature_A, feature_B, pairwise similarity score)
- Overlay visualization of the 30 top-neighbor edges on the Fig. 2B feature network showing connectivity beyond the k-medoids cluster

## How to apply

Load the msFeaST illustrative-example feature table and the precomputed pairwise spectral similarity matrix (MS2-based, e.g. ms2deepscore scores in the Pleurotus eryngii 0% vs 80% olive mill solid waste example) from the kevinmildau/msFeaST repository. Identify the top-prioritized differential feature from the globaltest-based feature-group testing (the Fig. 2B feature). Retrieve that feature's similarity row against all other features, rank in descending order, and take the top-30 nearest neighbors as an edge list of (feature_A, feature_B, similarity) records. Overlay these edges on the Fig. 2B feature network via the msFeaST_Dashboard_bundle.html dashboard (loading notebooks/data/omsw_pleurotus_ms2deepscore/dashboard_data.json) to verify that strong connectivity extends beyond the feature's own small highlighted cluster (cf. Supplementary Fig. S1); the edge weights let you judge connection strength between connected features.

## Related tools

- **msFeaST** (Supplies the feature table, precomputed spectral-similarity matrix, k-medoids feature grouping, and the interactive dashboard (msFeaST_Dashboard_bundle.html) on which the top-N neighbor edges are overlaid.) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Feature group-wise statistical testing used to identify the top-prioritized differential feature whose top-30 neighbor edges are extracted.)

## Examples

```
edges = sim.loc[feat].drop(feat).nlargest(30); pd.DataFrame({'feature_A': feat, 'feature_B': edges.index, 'similarity': edges.values}).to_csv('fig2b_top30_edges.csv', index=False)
```

## Evaluation signals

- Output CSV contains exactly 30 edges (one row per neighbor), matching the top-30 selection.
- Every edge carries a pairwise spectral-similarity score taken from the precomputed similarity matrix (values match the matrix entry for that feature pair).
- Neighbors are ranked in descending similarity order — the 30 selected edges are the feature's top-30 nearest neighbors.
- The overlaid connectivity pattern of the differential feature extends beyond its own small k-medoids spectral-similarity cluster, as reported in Section 3 / Supplementary Fig. S1.
- Feature identifiers in the edge list resolve to nodes present in the Fig. 2B network / dashboard_data.json.

## Limitations

- msFeaST pipeline preprocessing has been tested on macOS and Linux; Windows support was still being worked on, though the interactive dashboard works on desktop browsers regardless of OS.
- Conda/R path issues can silently install R dependencies (Matrix, survival, listenv, readr, tibble, dplyr, globaltest) into the wrong R path; terminals must be reopened after installing conda R.
- Windows users must run commands from the ANACONDA PROMPT, and certain terminals (e.g. vscode) may default into the wrong conda environment, breaking the setup.
- The skill depends on precomputed pairwise spectral similarity; it does not compute similarity scores itself.

## Evidence

- [abstract] msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information"
- [abstract] k-medoids grouping as proxy for structural similarity: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [readme] dashboard bundle and example data location: [evidence span withheld — anchor status: external_doc]
- [readme] example pipeline notebooks: [evidence span withheld — anchor status: external_doc]
