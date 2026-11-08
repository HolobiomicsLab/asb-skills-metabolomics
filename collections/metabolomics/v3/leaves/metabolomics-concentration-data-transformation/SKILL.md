---
name: metabolomics-concentration-data-transformation
description: Use when a post-normalization metabolite concentration table (.csv) is
  right-skewed or spans a wide dynamic range and an element-wise Log10, Log2, square
  root, or cube root transform is needed before scaling and PCA-based data-treatment
  assessment.
when_to_use_negative:
- The table has not yet been imputed and normalized — Viime performs transformation
  only after those steps, so applying it first breaks the data treatment order.
- The goal is scaling rather than transformation — autoscaling, Pareto, range, vast,
  and level scaling are a separate, later Viime step.
- The data are a multi-platform dataset needing fusion (concatenative, mid-level,
  or multi-block) rather than per-table transformation.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: Viime
  role: Provides the optional transformation step with four options (Log10, Log2,
    square root, cube root) inside its data treatment workflow, with interactive PCA
    feedback
  repo: girder/viime
- name: Pandas
  role: Parses the raw/normalized CSV concentration table and handles round-trip reading/writing
    of the transformed data frame
- name: R
  role: Implements the transformation (and other specialized processing) as functions
    in the viime R package exposed to the API server via OpenCPU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:714d18d4fb23c3075c7f5d8d3d2e6165039f4d456b4bea68d07cd14bc06bd4d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - R
  - Viime
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-concentration-data-transformation

## Summary

An optional element-wise data treatment step in Viime that applies Log10, Log2, square root, or cube root to a post-normalization metabolite concentration table to bring the distribution closer to normality and compress the dynamic range. It sits after imputation and normalization and before scaling, producing a transformed table ready for Viime's statistical analysis methods.

## When to use

Use when a post-normalization metabolite concentration table (.csv) is right-skewed or spans a wide dynamic range and an element-wise Log10, Log2, square root, or cube root transform is needed before scaling and PCA-based data-treatment assessment.

## When NOT to use

- The table has not yet been imputed and normalized — Viime performs transformation only after those steps, so applying it first breaks the data treatment order.
- The goal is scaling rather than transformation — autoscaling, Pareto, range, vast, and level scaling are a separate, later Viime step.
- The data are a multi-platform dataset needing fusion (concatenative, mid-level, or multi-block) rather than per-table transformation.

## Inputs

- Post-imputation, post-normalization metabolite concentration table (.csv) with primary ID, metadata/factor/group, and metabolite concentration columns

## Outputs

- Transformed .csv table with the same rows and columns as the input, where concentration values are replaced by the chosen element-wise transform (Log10, Log2, sqrt, or cbrt) and ID/metadata columns are retained

## How to apply

Load the concentration table (already imputed and normalized) from the uploaded CSV with Pandas, since Viime orders treatment as imputation -> normalization -> transformation -> scaling. Choose exactly one of the four supported element-wise transformations — Log10, Log2, square root, or cube root — based on the skew and dynamic range of the metabolite columns. Apply the transform only to metabolite concentration columns, preserving the primary ID and metadata columns. In Viime this step is implemented in R and invoked through the OpenCPU-backed REST service, where functions in the viime R package accept a CSV path and return a data frame. Write the transformed data frame back to CSV with the original row/column structure, and interactively check the effect with Viime's PCA score and loadings plots before proceeding to scaling.

## Related tools

- **Viime** (Provides the optional transformation step with four options (Log10, Log2, square root, cube root) inside its data treatment workflow, with interactive PCA feedback) — girder/viime
- **Pandas** (Parses the raw/normalized CSV concentration table and handles round-trip reading/writing of the transformed data frame)
- **R** (Implements the transformation (and other specialized processing) as functions in the viime R package exposed to the API server via OpenCPU)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('post_normalization.csv'); num_cols = [c for c in df.columns if c not in id_meta_cols]; df[num_cols] = df[num_cols].apply(np.log10); df.to_csv('transformed.csv', index=False)
```

## Evaluation signals

- Output is a valid CSV with exactly the same rows and columns as the input table.
- Each transformed concentration value equals the chosen transform applied element-wise to the input value (e.g., output == log10(input) for Log10).
- Primary ID and metadata/factor/group columns are unchanged by the transformation.
- The step was executed in order — after imputation and normalization, before scaling — in the Viime data treatment workflow.
- Downstream interactive PCA score/loadings plots on the transformed data can be used to judge whether the treatment improved group similarity/dissimilarity structure.

## Limitations

- The step is optional and offers only four element-wise choices (Log10, Log2, square root, cube root); no other transformation families are exposed.
- Log transforms are mathematically undefined for non-positive concentration values, so zero or negative entries must be handled beforehand (e.g., by imputation/normalization).
- The OpenCPU R-processing backend is described as work-in-progress; adding new R methods to the viime package requires rebuilding the Docker image.

## Evidence

- [abstract] The options in this step are Log10, Log2, square root and cube root.: "The options in this step are Log10, Log2, square root and cube root."
- [abstract] Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms.: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [readme] Functions contained in this package should accept a path to a CSV file as an argument and return a data frame.: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
