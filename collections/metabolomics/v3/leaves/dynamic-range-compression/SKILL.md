---
name: dynamic-range-compression
description: Use when a metabolomics concentration table (post-normalization, post-imputation)
  still shows skewed, heavy-tailed distributions that would violate normality assumptions
  in downstream PCA or group-comparison analyses.
when_to_use_negative:
- The input is raw, un-normalized data with missing values that still need Viime's
  automatic MCAR/MNAR imputation and normalization first
- The data are already approximately normal and the dynamic range is modest — Viime
  marks this step as optional and applying a transform unnecessarily can distort relative
  concentration patterns
- The goal is to modify ID, factor, group, or other metadata columns rather than metabolite
  concentration values
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Viime
  role: Hosts the optional data-transformation step (Log10, Log2, square root, cube
    root) as part of its data-treatment workflow between normalization and scaling
  repo: https://github.com/girder/viime
- name: Pandas
  role: Parses and holds the concentration table; the transformed data frame is written
    back to CSV for the scaling step
  repo: https://github.com/girder/viime
- name: R
  role: The transformation is programmed in R and exposed through Viime's OpenCPU-backed
    processing backend that accepts a CSV path and returns a data frame
  repo: https://github.com/girder/viime
- name: OpenCPU
  role: Provides the internal REST service bridging the Python/Flask backend and the
    R functions that perform the transformation
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:ed1eeb66081709c92b133ce9029bd2dd3ad80f6ddaf30ffa1d7fd3f76c1a0e33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Pandas
  - R
  - Viime
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# dynamic-range-compression

## Summary

An optional Viime data-treatment step that applies an element-wise transformation (Log10, Log2, square root, or cube root) to metabolomics concentration data, bringing the distribution closer to normality and compressing the dynamic range before scaling and analysis.

## When to use

Use when a metabolomics concentration table (post-normalization, post-imputation) still shows skewed, heavy-tailed distributions that would violate normality assumptions in downstream PCA or group-comparison analyses.

## When NOT to use

- The input is raw, un-normalized data with missing values that still need Viime's automatic MCAR/MNAR imputation and normalization first
- The data are already approximately normal and the dynamic range is modest — Viime marks this step as optional and applying a transform unnecessarily can distort relative concentration patterns
- The goal is to modify ID, factor, group, or other metadata columns rather than metabolite concentration values

## Inputs

- Normalized metabolomics concentration table in CSV format (uploaded to Viime via the .csv or .xlsx drag-and-drop interface), with a primary ID column, metadata/factor/group columns, and metabolite concentration columns

## Outputs

- Transformed concentration table in CSV format with the chosen transformation (Log10, Log2, square root, or cube root) applied element-wise to concentration columns, same rows and columns as the input, ready for the scaling step

## How to apply

Load the concentration table with Pandas after Viime's imputation and normalization steps have been completed, preserving the primary ID, factor, group, and metadata columns. Choose one of the four supported element-wise transforms — Log10, Log2, square root, or cube root — based on the observed spread and skew of the metabolite concentration values; the choice is user-configurable, so inspect the data first (e.g., via Viime's interactive PCA score and loadings plots used throughout data treatment). Apply the chosen transformation element-wise to the metabolite concentration columns only, leaving the primary ID and metadata columns untouched. Write the transformed data frame back to CSV so it can be fed into Viime's next data-treatment step (scaling) and subsequently into the analysis methods. Validate that the output CSV has the same rows and columns as the input and that each transformed value matches the chosen transform applied to the original concentration.

## Related tools

- **Viime** (Hosts the optional data-transformation step (Log10, Log2, square root, cube root) as part of its data-treatment workflow between normalization and scaling) — https://github.com/girder/viime
- **Pandas** (Parses and holds the concentration table; the transformed data frame is written back to CSV for the scaling step) — https://github.com/girder/viime
- **R** (The transformation is programmed in R and exposed through Viime's OpenCPU-backed processing backend that accepts a CSV path and returns a data frame) — https://github.com/girder/viime
- **OpenCPU** (Provides the internal REST service bridging the Python/Flask backend and the R functions that perform the transformation) — https://github.com/girder/viime

## Examples

```
viime-cli create-tables && flask run  # then in the Viime UI: upload the normalized CSV and select Log10 (or Log2 / sqrt / cbrt) in the data transformation step
```

## Evaluation signals

- Output CSV must have the identical row and column structure as the input table (same primary IDs and metabolite columns)
- Each transformed concentration value must equal the chosen transform applied element-wise to the corresponding input value (e.g., log10(x) for the Log10 option)
- Primary ID, factor, group, and other metadata columns must be left unchanged
- Post-transformation, concentration distributions should show reduced skew and compressed dynamic range, checkable via Viime's interactive PCA score and loadings plots before proceeding to scaling
- The transformed CSV must load cleanly into the subsequent Viime scaling step (autoscaling, Pareto, range, vast, or level scaling) without re-imputation

## Limitations

- The transformation step is optional and must be chosen manually — Viime does not auto-select Log10 vs Log2 vs square root vs cube root
- Element-wise transforms are applied only to concentration columns; there is no per-metabolite or per-group transformation option described
- The step sits strictly between imputation/normalization and scaling; applying it out of order (e.g., before imputation) is not supported
- Log10 and Log2 require positive concentration values; zero or negative entries (e.g., from prior imputation choices) would need care, though the article does not describe handling for these

## Evidence

- [abstract] data transformation options include Log10, Log2, square root and cube root to bring distribution closer to normality: "The options in this step are Log10, Log2, square root and cube root."
- [abstract] transformation comes after normalization and before scaling in Viime's data treatment workflow: "The first step in this process is data normalization."
- [abstract] Pandas handles raw file parsing and normalization in Viime's backend: "The backend leverages Pandas for raw file parsing and normalization."
- [readme] R functions exposed via OpenCPU accept a CSV path and return a data frame, used for CSV file processing: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [abstract] interactive PCA is used to guide data treatment choices: "A very important feature of the whole data treatment process is the interactive use of principal component analysis (PCA) to examine the similarity and dissimilarity of individual groups"
