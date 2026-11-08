---
name: spectral-cluster-annotation-mapping
description: Use when upstream msFeaST result tables (embeddings, k-medoids cluster
  labels, globaltest statistics, contrast mappings) must be joined per feature and
  serialized into the schema expected by the msFeaST_Dashboard_bundle.html interactive
  dashboard.
when_to_use_negative:
- Input is raw LC-MS/MS data or a quantification/metadata/spectral table that has
  not yet been processed by the msFeaST preprocessing and pipeline notebooks — the
  upstream clustering and globaltest steps must run first.
- You only want to inspect ready-made example data in the browser dashboard — then
  only msFeaST_Dashboard_bundle.html and an existing .json are needed and no integration
  is performed.
- Result tables do not share the m/z–retention-time feature key, since the join of
  embeddings, cluster labels, statistics, and contrast mappings depends on that identifier.
tools:
- name: msFeaST
  role: Workflow whose jupyter-notebook pipeline produces the .json results file consumed
    by the msFeaST_Dashboard_bundle.html visualization dashboard
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: Supplies the per-feature-group feature-set statistics that must be preserved
    per group and per contrast when aggregating records to the cluster level
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:64cf7f23d075a5b0de9f2cdd697395a3b9ac029b4c87499bff96b6387ccd0ea0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spectral-cluster-annotation-mapping

## Summary

Integrates msFeaST workflow outputs — per-feature t-SNE embedding coordinates, k-medoids spectral-similarity cluster assignments, per-feature and per-feature-group globaltest statistics, and contrast-specific feature-to-group mappings — into the single dashboard-ready msFeaST .json results file.

## When to use

Use when upstream msFeaST result tables (embeddings, k-medoids cluster labels, globaltest statistics, contrast mappings) must be joined per feature and serialized into the schema expected by the msFeaST_Dashboard_bundle.html interactive dashboard.

## When NOT to use

- Input is raw LC-MS/MS data or a quantification/metadata/spectral table that has not yet been processed by the msFeaST preprocessing and pipeline notebooks — the upstream clustering and globaltest steps must run first.
- You only want to inspect ready-made example data in the browser dashboard — then only msFeaST_Dashboard_bundle.html and an existing .json are needed and no integration is performed.
- Result tables do not share the m/z–retention-time feature key, since the join of embeddings, cluster labels, statistics, and contrast mappings depends on that identifier.

## Inputs

- per-feature t-SNE embedding coordinates table
- k-medoids cluster assignments per feature (spectral similarity-based groupings)
- per-feature and per-feature-group globaltest statistics
- contrast-specific feature-to-group mapping tables
- shared metabolomics feature identifier (m/z–retention-time feature key)

## Outputs

- single msfeast_results.json file in the dashboard-expected nested schema (embedding coordinates, cluster assignments, feature/group statistics, contrast mappings)

## How to apply

Load the upstream result tables with pandas: per-feature embedding coordinates, k-medoids cluster assignments (clusters proxy for structurally similar features), per-feature and per-feature-group globaltest statistics, and contrast-specific feature-to-group mappings. Join all tables on the shared metabolomics feature identifier (m/z–retention-time feature key) so each feature record carries its embedding position, cluster label, statistics, and contrast annotations. Aggregate feature-level records into the feature-group (cluster) level structure the dashboard requires, preserving the globaltest feature-set statistics per group and per contrast. Serialize the integrated structure with Python's json library into the single msfeast_results.json, nesting embeddings, cluster assignments, statistics, and contrast mappings in the dashboard-expected schema. Before handing the file to the dashboard, validate that it parses as valid JSON, contains all required top-level sections, and that every feature identifier appears in all sections.

## Related tools

- **msFeaST** (Workflow whose jupyter-notebook pipeline produces the .json results file consumed by the msFeaST_Dashboard_bundle.html visualization dashboard) — https://github.com/kevinmildau/msFeaST
- **globaltest** (Supplies the per-feature-group feature-set statistics that must be preserved per group and per contrast when aggregating records to the cluster level)

## Examples

```
import pandas as pd, json; integrated = embeddings.join(clusters).join(stats).join(contrasts); json.dump(integrated.to_dict(orient='index'), open('msfeast_results.json', 'w'))
```

## Evaluation signals

- msfeast_results.json parses as valid JSON without errors.
- The file contains the required top-level sections: embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings.
- Every feature identifier appears in all top-level sections of the .json.
- globaltest feature-set statistics are preserved per feature group and per contrast after aggregation to the cluster level.
- Loading the .json into msFeaST_Dashboard_bundle.html in a desktop browser succeeds and the dataview tab shows the loaded data.

## Limitations

- The msFeaST preprocessing and pipeline workflow has been tested on macOS and is expected to work identically on Linux; Windows support is untested/in progress (the browser dashboard itself works on any OS).
- Conda/rscript path caching problems can install R dependencies (Matrix, survival, listenv, readr, tibble, dplyr, globaltest 3.18) into the wrong R path; terminals must be reopened and the conda environment reactivated.
- The .json must exactly match the dashboard-expected schema; mis-nested or missing sections will prevent the interactive visualization from loading.

## Evidence

- [abstract] msFeaST interactive integration of experimental and structural information: "Hence, msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and"
- [readme] README: pipeline produces json file for dashboard: [evidence span withheld — anchor status: external_doc]
- [readme] README: dashboard bundle plus .json is all that is needed to view data: [evidence span withheld — anchor status: external_doc]
- [readme] README: dashboard works in browsers on any OS: [evidence span withheld — anchor status: external_doc]
