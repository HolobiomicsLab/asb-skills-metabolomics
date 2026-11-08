---
name: lc-msms-feature-set-data-integration
description: Use when a completed msFeaST pipeline has produced embedding coordinates,
  k-medoids cluster labels, and globaltest statistics per feature, and these must
  be joined on the m/z–retention-time feature key and serialized into the msFeaST_Dashboard_bundle.html-compatible
  .json schema.
when_to_use_negative:
- The upstream pipeline has not yet produced clustering, globaltest statistics, or
  embedding tables — joining incomplete tables would yield a .json missing required
  top-level sections
- The goal is interactive exploration without running the pipeline; then only msFeaST_Dashboard_bundle.html
  plus a pre-made .json is needed and no integration step should be re-run
- Data are not LC-MS/MS untargeted metabolomics features keyed by m/z–retention-time,
  so the join key and feature-group structure would not match the dashboard schema
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Python/R/jupyter-notebook workflow whose pipeline joins the result tables
    and serializes the integrated structure into the dashboard-expected .json; the
    msFeaST_Dashboard_bundle.html then consumes that file
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Supplies the per-feature-group feature-set statistics (installed at Bioconductor
    3.18 / globaltest 5.56.0) that must be preserved per group and per contrast during
    integration
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:0f7adc6b1fa7ec58de264e907431b489a8cacf638a6f167ba91e328cbcf5a42a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lc-msms-feature-set-data-integration

## Summary

Integrates LC-MS/MS untargeted metabolomics results (per-feature t-SNE embedding coordinates, k-medoids cluster assignments, globaltest feature-set statistics, and contrast mappings) into a single dashboard-ready .json file that drives the msFeaST interactive browser-based visualization.

## When to use

Use when a completed msFeaST pipeline has produced embedding coordinates, k-medoids cluster labels, and globaltest statistics per feature, and these must be joined on the m/z–retention-time feature key and serialized into the msFeaST_Dashboard_bundle.html-compatible .json schema.

## When NOT to use

- The upstream pipeline has not yet produced clustering, globaltest statistics, or embedding tables — joining incomplete tables would yield a .json missing required top-level sections
- The goal is interactive exploration without running the pipeline; then only msFeaST_Dashboard_bundle.html plus a pre-made .json is needed and no integration step should be re-run
- Data are not LC-MS/MS untargeted metabolomics features keyed by m/z–retention-time, so the join key and feature-group structure would not match the dashboard schema

## Inputs

- Per-feature t-SNE embedding coordinate table (pandas-loadable)
- k-medoids cluster assignment table keyed by m/z–retention-time feature identifier
- Per-feature and per-feature-group globaltest statistics tables
- Contrast-specific feature-to-group mapping tables
- Quantification table, metadata table, and spectral data from the msFeaST preprocessing/pipeline notebooks

## Outputs

- A single msFeaST results .json file (e.g. notebooks/data/omsw_pleurotus_ms2deepscore/dashboard_data.json) with nested embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings
- A dashboard-ready artifact loadable into msFeaST_Dashboard_bundle.html in a desktop browser

## How to apply

Load the upstream result tables with pandas: per-feature embedding coordinates, k-medoids cluster assignments, per-feature and per-feature-group globaltest statistics, and contrast-specific feature-to-group mappings. Join all tables on the shared metabolomics feature identifier (the m/z–retention-time feature key) so each record carries its embedding position, cluster label, statistics, and contrast annotations. Aggregate feature-level records into the feature-group (cluster) level structure the dashboard expects, preserving globaltest feature-set statistics per group and per contrast. Serialize the integrated structure with Python's json library into a single results .json (e.g. dashboard_data.json), nesting embeddings, cluster assignments, statistics, and contrast mappings in the dashboard-expected schema. The msFeaST jupyter-notebook pipeline produces this json file that can be interactively explored in the interactive dashboard.

## Related tools

- **msFeaST** (Python/R/jupyter-notebook workflow whose pipeline joins the result tables and serializes the integrated structure into the dashboard-expected .json; the msFeaST_Dashboard_bundle.html then consumes that file) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Supplies the per-feature-group feature-set statistics (installed at Bioconductor 3.18 / globaltest 5.56.0) that must be preserved per group and per contrast during integration)

## Examples

```
conda activate msfeast_environment; jupyter-notebook msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- msfeast_results.json parses as valid JSON (e.g. json.load succeeds without error)
- The file contains the required top-level sections: embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings
- Every feature identifier (m/z–retention-time key) appears in all sections — no feature is present in embeddings but missing from statistics or contrast mappings
- globaltest feature-set statistics are preserved per feature group and per contrast after aggregation from feature-level records
- The .json loads into msFeaST_Dashboard_bundle.html in a desktop browser (firefox, chrome, edge, safari) and the dataview tab shows the loaded data

## Limitations

- The pipeline is tested on macOS and expected to work identically on Linux; Windows support is untested and requires the Anaconda Prompt
- R packages are installed via remotes/BiocManager rather than conda; rscript path caching after conda R installation can cause packages to install in the wrong R path (mitigate by reopening the terminal and reactivating msfeast_environment)
- The dashboard bundle alone works only for inspecting pre-processed example files — it cannot regenerate the integrated .json without the Python/R pipeline
- R installation compilation steps may require several minutes, and specific pinned versions are required (R 4.3.3, Matrix 1.6-5, survival 3.5-8, globaltest 3.18/5.56.0)

## Evidence

- [other] The msFeaST workflow integrates all of its results into a .json formatted file that can be read by the msFeaST visualization dashboard: [evidence span withheld — anchor status: not_found]
- [readme] The jupyter-notebook pipeline produces the a text file in json format that can be interactively explored in the interactive dashboard.: [evidence span withheld — anchor status: external_doc]
- [readme] The interactive visualization dashboard works regardless of os on desktop browsers (e.g., firefox, chrome, edge, safari): [evidence span withheld — anchor status: external_doc]
- [readme] To make use of your own data, change the data filepath arguments to your own data file location and run the pipeline.: [evidence span withheld — anchor status: external_doc]
- [abstract] msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and"
- [readme] please download the *msFeaST_Dashboard_bundle.html* and the ready made data from notebooks\data\omsw_pleurotus_ms2deepscore\dashboard_data.json: [evidence span withheld — anchor status: external_doc]
