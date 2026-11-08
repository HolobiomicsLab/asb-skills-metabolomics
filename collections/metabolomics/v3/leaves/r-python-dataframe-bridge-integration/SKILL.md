---
name: r-python-dataframe-bridge-integration
description: Use when a Python-based metabolomics pipeline (pandas DataFrames) must
  call R statistical routines — e.g. running the multivariate ANOVA + Tukey post-hoc
  test on metabolite concentration tables in Viime.
when_to_use_negative:
- Input data is not already a CSV concentration table (e.g. raw vendor files or .xlsx
  uploads that have not been parsed/normalized by Pandas on the backend).
- The analysis is entirely doable in the Python stack — invoking OpenCPU adds an extra
  service and image rebuild for no benefit.
- A function needs to be added but the OpenCPU Docker image has not been rebuilt —
  new R methods will not be exposed until that happens.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: OpenCPU
  role: Secondary REST service that internally exposes the R processing functions
    to the Flask API server
  repo: https://github.com/girder/viime
- name: Flask
  role: Web-framework backend that calls the OpenCPU service to execute R routines
    on CSV data
  repo: https://github.com/girder/viime
- name: Pandas
  role: Parses raw CSVs into DataFrames and receives converted R data frame results
  repo: https://github.com/girder/viime
- name: R
  role: Statistics stack executing specialized algorithms (e.g. multivariate ANOVA
    with post-hoc Tukey) via the custom viime R package
  repo: https://github.com/girder/viime
- name: Viime
  role: The custom R package (devops/viime) whose functions are exposed through OpenCPU;
    also the overall web application orchestrating the bridge
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:b53cbb2c9df3cc4ed981da29bf0c5b8588f07e2caaebb1e157a2877b02e016a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - OpenCPU
  - Pandas
  - R
  - Viime
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# r-python-dataframe-bridge-integration

## Summary

Bridges the Python (Flask/Pandas) backend of Viime with the R statistics stack by exposing a custom R package of CSV-processing functions through an OpenCPU REST service, so specialized statistical algorithms (e.g. ANOVA with post-hoc Tukey) can run on R while the frontend works with pandas DataFrames.

## When to use

Use when a Python-based metabolomics pipeline (pandas DataFrames) must call R statistical routines — e.g. running the multivariate ANOVA + Tukey post-hoc test on metabolite concentration tables in Viime.

## When NOT to use

- Input data is not already a CSV concentration table (e.g. raw vendor files or .xlsx uploads that have not been parsed/normalized by Pandas on the backend).
- The analysis is entirely doable in the Python stack — invoking OpenCPU adds an extra service and image rebuild for no benefit.
- A function needs to be added but the OpenCPU Docker image has not been rebuilt — new R methods will not be exposed until that happens.

## Inputs

- CSV file path to a preprocessed metabolomics concentration table (samples as rows, metabolite concentration columns, factor/group column)
- pandas DataFrame on the Python/Flask backend

## Outputs

- R data frame returned by the OpenCPU-exposed function, converted back to a pandas DataFrame
- results table (e.g. ANOVA p-values plus per-pairwise-group Tukey p-values) keyed by metabolite

## How to apply

Package the R functions in a custom R package (devops/viime in the Viime repo) where each function accepts a path to a CSV file and returns a data frame. Serve this package through an OpenCPU REST instance (built as a Docker container via devops Dockerfile, or via docker-compose with docker_opencpu_1), so the Flask backend can call it. In the Python side, viime/opencpu.py handles communication between pandas DataFrames and R data frames — sending the preprocessed metabolite concentration CSV (samples as rows, metabolite columns, factor/group column) to R, running the R routine (e.g. multivariate ANOVA plus post-hoc Tukey for two or more groups), and converting the returned R data frame back into pandas for downstream use. To expose additional methods to the API server, add the function to the viime R package and rebuild the Docker image.

## Related tools

- **OpenCPU** (Secondary REST service that internally exposes the R processing functions to the Flask API server) — https://github.com/girder/viime
- **Flask** (Web-framework backend that calls the OpenCPU service to execute R routines on CSV data) — https://github.com/girder/viime
- **Pandas** (Parses raw CSVs into DataFrames and receives converted R data frame results) — https://github.com/girder/viime
- **R** (Statistics stack executing specialized algorithms (e.g. multivariate ANOVA with post-hoc Tukey) via the custom viime R package) — https://github.com/girder/viime
- **Viime** (The custom R package (devops/viime) whose functions are exposed through OpenCPU; also the overall web application orchestrating the bridge) — https://github.com/girder/viime

## Examples

```
cd devops && docker build -t viime . && docker run --rm --name viime -p 8004:8004 viime
```

## Evaluation signals

- R functions in the custom package must accept a CSV file path and return a data frame.
- A round-trip test (pandas → R → pandas) should preserve row/column identity for a known metabolite table.
- For the ANOVA+Tukey use case, verify the output CSV contains a p-value entry for the ANOVA and for each pairwise Tukey comparison of every tested metabolite, with all reported p-values in [0, 1].
- After adding an R method, the Docker image must be rebuilt; a stale image is a concrete failure signal if the new function is not reachable via the API.
- docker-compose should yield two containers (docker_backend_1 and docker_opencpu_1); the OpenCPU service must be reachable on the configured port (e.g. 8004).

## Limitations

- The backend communication code at viime/opencpu.py is described as work-in-progress in the README.
- Adding a new R method requires rebuilding the OpenCPU Docker image; changes are not picked up live.
- The OpenCPU container may terminate with SIGWINCH if the window position/size changes; running without a pseudo-terminal and keeping stdin open avoids this.
- Only functions in the custom viime R package are exposed; other R code must be wrapped the same way.

## Evidence

- [readme] This installation includes a custom R package called viime, which contains all functions that are exposed to the API server for CSV file processing. Functions contained in this package should accept a path to a CSV file as an argument and return a data frame.: [evidence span withheld — anchor status: not_found]
- [readme] The (work-in-progress) backend code at viime/opencpu.py handles the communication between pandas and R data frames.: [evidence span withheld — anchor status: not_found]
- [abstract] The R-python integration is provided by a secondary REST service exposed internally via OpenCPU.: "The R-python integration is provided by a secondary REST service exposed internally via OpenCPU."
- [abstract] Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms.: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [abstract] Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively.: "Univariate analyses using the Wilcoxon rank sum test and multivariate ANOVA can be carried out on data with two or more groups, respectively."
