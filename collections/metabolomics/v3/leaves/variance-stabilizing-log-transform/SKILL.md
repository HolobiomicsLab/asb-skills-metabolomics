---
name: variance-stabilizing-log-transform
description: Use when a metabolite concentration table (post-imputation and post-normalization,
  from a .csv or .xlsx upload) is right-skewed or spans a wide dynamic range and needs
  an element-wise log/power transform before scaling and PCA-based treatment selection.
when_to_use_negative:
- The concentration table still contains missing values or has not been normalized
  — imputation (e.g., missForest/impute) and normalization precede transformation
  in Viime's workflow
- Only common statistical algorithms are needed — Viime implements those with Scikit-learn
  rather than the specialized R transformation functions
- The data are already approximately normal with a compressed dynamic range — the
  transformation step in Viime is optional
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: Web application whose data treatment workflow exposes the transformation step
    with Log10, Log2, square root, and cube root options
  repo: https://github.com/girder/viime
- name: Pandas
  role: Used to parse the CSV concentration table and apply the element-wise transformation
    while preserving row/column structure
- name: R
  role: Hosts the custom viime R package whose CSV-processing functions implement
    the specialized transformations, exposed via OpenCPU
  repo: https://github.com/girder/viime
- name: OpenCPU
  role: Internal REST service providing the R–Python integration through which transformation
    functions are called from the backend
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:b87f37f901585b8aa7fa7e594ce84c0c9a3f9d441d90cea0056c5bdb1d50221d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - R
  - Viime
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# variance-stabilizing-log-transform

## Summary

Element-wise data transformation step from Viime's data treatment workflow that applies Log10, Log2, square root, or cube root to metabolite concentration columns, bringing the distribution closer to normality and compressing the dynamic range before scaling.

## When to use

Use when a metabolite concentration table (post-imputation and post-normalization, from a .csv or .xlsx upload) is right-skewed or spans a wide dynamic range and needs an element-wise log/power transform before scaling and PCA-based treatment selection.

## When NOT to use

- The concentration table still contains missing values or has not been normalized — imputation (e.g., missForest/impute) and normalization precede transformation in Viime's workflow
- Only common statistical algorithms are needed — Viime implements those with Scikit-learn rather than the specialized R transformation functions
- The data are already approximately normal with a compressed dynamic range — the transformation step in Viime is optional

## Inputs

- Normalized metabolomics concentration table as CSV (rows = samples, columns = metabolite concentrations plus a designated primary ID, factor/group, and metadata columns)
- User selection of one transformation: Log10, Log2, square root, or cube root

## Outputs

- Transformed data frame written back to CSV with the same row and column structure as the input, concentration values element-wise transformed and ID/metadata columns unchanged
- Transformed dataset ready for input into the data scaling step and analysis methods

## How to apply

Load the normalized metabolomics concentration table from the uploaded CSV with Pandas, preserving the primary ID, factor, group, and metadata columns. Choose one of Viime's four supported element-wise transformations — Log10, Log2, square root, or cube root — via the interactive data treatment interface (or programmatically via the R processing functions exposed through OpenCPU). Apply the chosen function element-wise to the metabolite concentration columns only, leaving ID and metadata columns untouched. In Viime's backend, specialized transformations are implemented as R package functions that accept a path to a CSV file and return a data frame, with viime/opencpu.py handling communication between pandas and R data frames. The transformed table is then ready for the scaling step (autoscaling, Pareto, range, vast, or level scaling) and downstream analysis; the choice of transform can be guided by examining interactive PCA score and loadings plots.

## Related tools

- **Viime** (Web application whose data treatment workflow exposes the transformation step with Log10, Log2, square root, and cube root options) — https://github.com/girder/viime
- **Pandas** (Used to parse the CSV concentration table and apply the element-wise transformation while preserving row/column structure)
- **R** (Hosts the custom viime R package whose CSV-processing functions implement the specialized transformations, exposed via OpenCPU) — https://github.com/girder/viime
- **OpenCPU** (Internal REST service providing the R–Python integration through which transformation functions are called from the backend)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('normalized.csv'); df[conc_cols] = df[conc_cols].apply(np.log10); df.to_csv('transformed.csv', index=False)
```

## Evaluation signals

- Output CSV has the same rows and columns as the input table, with primary ID and metadata columns unchanged
- Every transformed concentration value equals the chosen function (log10, log2, sqrt, or cbrt) applied element-wise to the corresponding input value
- Round-tripping the output (e.g., exponentiating after a log transform) recovers the pre-transformation values
- Transformed table loads successfully into the subsequent scaling step (autoscaling, Pareto, range, vast, or level scaling) and PCA visualization
- R-function route returns a data frame with the expected shape when given a path to the input CSV

## Limitations

- Only four transformations are supported (Log10, Log2, square root, cube root); other variance-stabilizing transforms are not offered
- The R processing route requires a separately deployed OpenCPU Docker instance (devops directory) and rebuilding the image to add new functions
- The opencpu.py backend code bridging pandas and R data frames is described in the README as work-in-progress

## Evidence

- [abstract] Transformation options limited to four named element-wise functions: "The options in this step are Log10, Log2, square root and cube root."
- [abstract] Data processing split between Scikit-learn and specialized R packages: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [readme] Custom R package functions accept a CSV path and return a data frame: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [abstract] PCA is used to evaluate data treatment choices interactively: "A very important feature of the whole data treatment process is the interactive use of principal component analysis (PCA)"
