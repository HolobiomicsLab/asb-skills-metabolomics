---
name: metabolomics-feature-metadata-joining
description: Use when upstream msFeaST pipeline tables (embedding coordinates, cluster
  labels, globaltest statistics, contrast mappings) must be joined and serialized
  into a dashboard-ready msFeaST results .json file.
when_to_use_negative:
- The upstream workflow has not yet produced clustering, statistical testing, or embedding
  outputs — there is nothing to integrate yet
- The target visualization is not the msFeaST browser dashboard, so the .json dashboard
  schema is not the required interchange format
- Input is already a single validated dashboard .json file — re-joining would risk
  breaking the dashboard-expected schema
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Workflow whose pipeline produces the joined results .json file and whose dashboard
    consumes it for interactive visualization
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: R package supplying the per-feature-group statistical testing results that
    are preserved per group and per contrast during the join and aggregation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:21aa52205473f9693366ce7f82508fbaae10d1918fe1e8f7a926fb4a51aa2202
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-feature-metadata-joining

## Summary

Integrates per-feature embedding coordinates, k-medoids cluster assignments, globaltest statistics, and contrast-specific feature-to-group mappings from an LC-MS/MS untargeted metabolomics workflow into the single msFeaST results .json file consumed by the browser-based interactive dashboard.

## When to use

Use when upstream msFeaST pipeline tables (embedding coordinates, cluster labels, globaltest statistics, contrast mappings) must be joined and serialized into a dashboard-ready msFeaST results .json file.

## When NOT to use

- The upstream workflow has not yet produced clustering, statistical testing, or embedding outputs — there is nothing to integrate yet
- The target visualization is not the msFeaST browser dashboard, so the .json dashboard schema is not the required interchange format
- Input is already a single validated dashboard .json file — re-joining would risk breaking the dashboard-expected schema

## Inputs

- Per-feature embedding coordinate table (e.g., t-SNE coordinates per m/z–retention-time feature)
- k-medoids cluster assignment table mapping features to structural-similarity-based feature groups
- Per-feature and per-feature-group globaltest statistics table
- Contrast-specific feature-to-group mapping table
- Shared metabolomics feature identifier key (m/z–retention-time feature)

## Outputs

- Single msFeaST results .json file (msfeast_results.json) nesting embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings
- Dashboard-loadable data file loadable in msFeaST_Dashboard_bundle.html (e.g., dashboard_data.json)

## How to apply

Load the upstream result tables with pandas: per-feature t-SNE embedding coordinates, k-medoids cluster assignments, per-feature and per-feature-group globaltest statistics, and contrast-specific feature-to-group mappings. Join all tables on the shared m/z–retention-time feature identifier so each feature record carries its embedding position, cluster label, statistics, and contrast annotations. Aggregate feature-level records into the feature-group (cluster) level structure required by the dashboard, preserving the globaltest feature-set statistics per group and per contrast. Serialize the integrated structure with Python's json library into a single msfeast_results.json, nesting embeddings, cluster assignments, statistics, and contrast mappings in the dashboard-expected schema. Validate that the file parses as valid JSON and contains all required top-level sections with consistent feature identifiers before handing it to the msFeaST_Dashboard_bundle.html.

## Related tools

- **msFeaST** (Workflow whose pipeline produces the joined results .json file and whose dashboard consumes it for interactive visualization) — https://github.com/kevinmildau/msFeaST
- **globaltest** (R package supplying the per-feature-group statistical testing results that are preserved per group and per contrast during the join and aggregation)

## Examples

```
conda activate msfeast_environment && jupyter-notebook  # run msfeast_pipeline_mushroom_type_comparison.ipynb to produce the results .json
```

## Evaluation signals

- msfeast_results.json parses as valid JSON without errors
- The file contains the required top-level sections: embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings
- Every feature identifier appears in all required sections (no feature present in one section but missing in another)
- Feature-group level records preserve the globaltest feature-set statistics per group and per contrast
- The .json loads successfully in msFeaST_Dashboard_bundle.html in a desktop browser and is inspectable in the dataview tab

## Limitations

- The pipeline (and thus the join/export step) is tested on macOS and expected to work identically on Linux; Windows support is still being worked on (untested, requires ANACONDA PROMPT)
- The interactive dashboard works only on desktop browsers (e.g., firefox, chrome, edge, safari), so mobile/browser-constrained environments are unsupported
- globaltest and its R dependency stack must be installed at pinned versions (globaltest 5.56.0 via Bioconductor 3.18, R 4.3.3); rscript path caching issues can install packages into the wrong R environment and break the pipeline

## Evidence

- [other] Load the upstream result tables with pandas: per-feature embedding coordinates, k-medoids cluster assignments, per-feature and per-feature-group globaltest statistics, and contrast-specific feature-to-group mappings.: [evidence span withheld — anchor status: not_found]
- [readme] The jupyter-notebook pipeline produces the a text file in json format that can be interactively explored in the interactive dashboard.: [evidence span withheld — anchor status: external_doc]
- [abstract] Hence, msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information"
- [readme] please download the *msFeaST_Dashboard_bundle.html* and the ready made data from notebooks\data\omsw_pleurotus_ms2deepscore\dashboard_data.json.: [evidence span withheld — anchor status: external_doc]
- [abstract] Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package: "Spectral clustering done in this way allows for feature group-wise statistical testing using the globaltest package"
