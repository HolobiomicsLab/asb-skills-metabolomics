---
name: contrast-level-statistics-integration
description: Use when msFeaST pipeline outputs need to be integrated and serialized
  into a single .json file for interactive dashboard consumption.
when_to_use_negative:
- When the goal is only to inspect pre-processed example files in the dashboard —
  the README states the msFeaST_Dashboard_bundle.html is the only file needed alongside
  the .json, with no dependencies installed
tools:
- name: msFeaST
  role: Pipeline module whose output results are integrated into the single .json
    file for dashboard visualization
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:4c80ef645cb1f667882f8775e02e3d4c8642fdbdf6f5954d27d2708f4c3cb361
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# contrast-level-statistics-integration

## Summary

Integrates feature-group statistical results from msFeaST into a single .json results file consumable by the interactive msFeaST visualization dashboard.

## When to use

Use when msFeaST pipeline outputs need to be integrated and serialized into a single .json file for interactive dashboard consumption.

## When NOT to use

- When the goal is only to inspect pre-processed example files in the dashboard — the README states the msFeaST_Dashboard_bundle.html is the only file needed alongside the .json, with no dependencies installed

## Inputs

- Upstream result tables with pandas (per-feature embedding coordinates, k-medoids cluster assignments, per-feature and per-feature-group globaltest statistics, contrast-specific feature-to-group mappings)

## Outputs

- msfeast_results.json (single .json formatted file readable by the msFeaST interactive visualization dashboard)

## How to apply

Integrates per-feature embedding coordinates, k-medoids cluster assignments, per-feature and per-feature-group globaltest statistics, and contrast-specific feature-to-group mappings by joining tables on the shared metabolomics feature identifier. The workflow loads upstream result tables with pandas, joins them on the shared feature identifier, aggregates feature-level records into feature-group level structure required by the dashboard, then serializes the integrated structure with Python's json library. Validation ensures the output parses as valid JSON and contains required top-level sections.

## Related tools

- **msFeaST** (Pipeline module whose output results are integrated into the single .json file for dashboard visualization) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment; jupyter-notebook
```

## Evaluation signals

- Verify msfeast_results.json parses as valid JSON
- Check the output contains required top-level sections: embedding coordinates, cluster assignments, feature/group statistics, contrast mappings
- Confirm every feature identifier appears in all sections before handing the file to the dashboard
- Ensure the integrated structure matches the dashboard-expected schema (feature-group level structure preserving globaltest feature-set statistics per group and per contrast)

## Limitations

- Windows support for the pre-processing and pipeline workflow is currently being worked on; the interactive dashboard itself works on desktop browsers across operating systems
- R package installation uses remotes and BiocManager instead of conda because the conda R package environment is not yet reliable for required packages
- The rscript command may cache a faulty R path if run from a terminal prior to the conda installation of R; close and reopen the terminal and re-activate the conda environment to avoid installing packages into the wrong R path

## Evidence

- [full_text] Joining and serializing into a single JSON result: [evidence span withheld — anchor status: not_found]
- [full_text] Dashboard is local browser-based JavaScript tool: [evidence span withheld — anchor status: not_found]
- [methods] Validation of JSON output: [evidence span withheld — anchor status: not_found]
- [readme] Dashboard needs no dependencies: [evidence span withheld — anchor status: external_doc]
