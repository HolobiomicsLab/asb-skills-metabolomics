---
name: json-results-serialization-for-dashboard
description: Use when you have finished the msFeaST pipeline notebooks (quantification
  table, metadata, spectral data processed) and need to produce the dashboard-loadable
  .json results file from the upstream result tables.
when_to_use_negative:
- The pipeline has not yet produced the clustering and globaltest result tables —
  there is nothing to integrate and the dashboard would load an incomplete structure.
- 'The goal is purely to inspect pre-processed example files in the dashboard: in
  that case the README states the msFeaST_Dashboard_bundle.html plus the ready-made
  dashboard_data.json are the only files needed and no Python serialization is required.'
- The user wants a static figure or report rather than interactive dashboard exploration
  — the msFeaST dashboard is the target consumer of this .json, not general-purpose
  plotting tools.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: provides the python module and jupyter-notebook pipeline whose output tables
    are integrated and serialized into the dashboard .json; also ships the msFeaST_Dashboard_bundle.html
    consumer
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: produces the feature-group-wise statistics that must be preserved per group
    and per contrast when serializing into the dashboard .json
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:bc0f39fd3dc67c86c35daac9295b222d093ae4c5a9f88c54995a82542b27954a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# json-results-serialization-for-dashboard

## Summary

Serialize the msFeaST pipeline's integrated clustering, globaltest statistics, and embedding results into a single .json file that the msFeaST interactive visualization dashboard (a browser-based JavaScript bundle) can load for exploratory feature prioritization.

## When to use

Use when you have finished the msFeaST pipeline notebooks (quantification table, metadata, spectral data processed) and need to produce the dashboard-loadable .json results file from the upstream result tables.

## When NOT to use

- The pipeline has not yet produced the clustering and globaltest result tables — there is nothing to integrate and the dashboard would load an incomplete structure.
- The goal is purely to inspect pre-processed example files in the dashboard: in that case the README states the msFeaST_Dashboard_bundle.html plus the ready-made dashboard_data.json are the only files needed and no Python serialization is required.
- The user wants a static figure or report rather than interactive dashboard exploration — the msFeaST dashboard is the target consumer of this .json, not general-purpose plotting tools.

## Inputs

- per-feature embedding coordinates (e.g. t-SNE) from the msFeaST pipeline
- k-medoids cluster assignments per LC-MS/MS feature
- globaltest feature-set statistics per feature group and per contrast
- contrast-specific feature-to-group mappings
- quantification table, metadata table, and processed spectral data from the preprocessing notebook

## Outputs

- a single .json formatted results file (e.g. dashboard_data.json / msfeast_results.json) loadable by the msFeaST interactive dashboard
- a JSON structure nesting embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings

## How to apply

After the msFeaST jupyter-notebook pipeline (msfeast_pipeline_mushroom_type_comparison.ipynb) has computed k-medoids spectral-similarity feature groupings, globaltest feature-set statistics, and the t-SNE-style embeddings, load the per-feature result tables with pandas and join them on the shared metabolomics feature identifier (m/z–retention-time key) so each record carries its embedding position, cluster label, statistics, and contrast annotations. Aggregate feature-level records into the feature-group (cluster) level structure the dashboard expects, preserving per-group and per-contrast globaltest statistics. Serialize the integrated structure with Python's json library into a single msfeast_results.json (the README's example is dashboard_data.json), nesting embeddings, cluster assignments, statistics, and contrast mappings in the dashboard-expected schema. The rationale is that the dashboard bundle (msFeaST_Dashboard_bundle.html) is a self-contained JavaScript tool that reads only this .json, so the file must be valid JSON and schema-complete for the interactive integration of semi-quantitative experimental information with mass-spectral structural similarity information.

## Related tools

- **msFeaST** (provides the python module and jupyter-notebook pipeline whose output tables are integrated and serialized into the dashboard .json; also ships the msFeaST_Dashboard_bundle.html consumer) — https://github.com/kevinmildau/msFeaST
- **globaltest** (produces the feature-group-wise statistics that must be preserved per group and per contrast when serializing into the dashboard .json)

## Examples

```
conda activate msfeast_environment; jupyter-notebook msfeast_pipeline_mushroom_type_comparison.ipynb  # produces the .json results file
```

## Evaluation signals

- msfeast_results.json parses as valid JSON (e.g. python -m json.tool or json.load succeeds).
- The JSON contains the required top-level sections: embedding coordinates, cluster assignments, feature/group statistics, and contrast mappings.
- Every metabolomics feature identifier appears in all sections of the .json (no orphan features in embeddings but missing from statistics or clusters).
- globaltest statistics per feature group and per contrast match the values in the upstream result tables before serialization.
- The dashboard bundle (msFeaST_Dashboard_bundle.html) loads the file without error and the dataview tab shows the loaded data.

## Limitations

- R installation within the conda environment is a prerequisite for running the pipeline whose outputs are serialized; rscript path caching issues can cause wrong-package installs, which would corrupt upstream results before serialization.
- Windows support for the pipeline is still being worked on (UNTESTED), so serialization workflows on Windows may need the ANACONDA PROMPT and may not be validated.
- The dashboard works on desktop browsers only (firefox, chrome, edge, safari); mobile or unsupported browsers cannot consume the serialized .json.

## Evidence

- [full_text] workflow: ... Serialize the integrated structure with Python's json library into the single msFeaST results .json file, nesting embeddings, cluster assignments, statistics, and contrast mappings in the dashboard-expected schema.: [evidence span withheld — anchor status: not_found]
- [full_text] Validation: verify msfeast_results.json parses as valid JSON, contains the required top-level sections (embedding coordinates, cluster assignments, feature/group statistics, contrast mappings), and that every feature identifier appears in all sections: [evidence span withheld — anchor status: not_found]
- [readme] The jupyter-notebook pipeline produces the a text file in json format that can be interactively explored in the interactive dashboard.: [evidence span withheld — anchor status: external_doc]
- [readme] please download the msFeaST_Dashboard_bundle.html and the ready made data from notebooks\data\omsw_pleurotus_ms2deepscore\dashboard_data.json: [evidence span withheld — anchor status: external_doc]
- [abstract] Hence, msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information, enhancing the prioritization of features and: "msFeaST provides interactive integration of the semi-quantitative experimental information with mass-spectral structural similarity information"
