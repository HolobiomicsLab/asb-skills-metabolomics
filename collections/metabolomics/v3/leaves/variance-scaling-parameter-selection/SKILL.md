---
name: variance-scaling-parameter-selection
description: Use when a preprocessed metabolomics data table (already normalized and
  log/sqrt/cube-root transformed) still spans a large dynamic range across metabolites
  and must be variance-scaled before statistical analysis or PCA-guided data treatment
  review.
when_to_use_negative:
- The data table has not yet been through Viime's earlier treatment steps (error correction/imputation,
  normalization, and transformation such as Log10/Log2/square root/cube root) — scaling
  is the final step, not a substitute for them.
- The goal is missing-data imputation (MNAR/MCAR classification via missForest/impute)
  or normalization to a reference sample — those precede scaling in Viime's workflow.
- The task is multi-platform data integration (simple/mid-level/multi-block fusion)
  rather than per-column variance scaling of a single table.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Scikit-learn
  role: Provides the common statistical algorithms used by Viime's backend, including
    the scaling utilities applied per metabolite column for the five scaling options
- name: Pandas
  role: Used by Viime's backend to load and manipulate the metabolomics data table
    (CSV) that is scaled and written back out
- name: Viime
  role: The application whose final data treatment step performs scaling, offering
    autoscaling, Pareto, range, vast, and level scaling, evaluated via interactive
    PCA plots
  repo: https://github.com/girder/viime
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:daf2dd6beb1938c35b3d755c539c023cd9f0f6f2a25896ae404f55798e334a45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# variance-scaling-parameter-selection

## Summary

The final data treatment step in Viime, which addresses the large dynamic range of metabolomics data by scaling the variance of each metabolite column, with options including autoscaling, Pareto scaling, range scaling, vast scaling and level scaling.

## When to use

Use when a preprocessed metabolomics data table (already normalized and log/sqrt/cube-root transformed) still spans a large dynamic range across metabolites and must be variance-scaled before statistical analysis or PCA-guided data treatment review.

## When NOT to use

- The data table has not yet been through Viime's earlier treatment steps (error correction/imputation, normalization, and transformation such as Log10/Log2/square root/cube root) — scaling is the final step, not a substitute for them.
- The goal is missing-data imputation (MNAR/MCAR classification via missForest/impute) or normalization to a reference sample — those precede scaling in Viime's workflow.
- The task is multi-platform data integration (simple/mid-level/multi-block fusion) rather than per-column variance scaling of a single table.

## Inputs

- normalized and transformed metabolomics data table (.csv or .xlsx, e.g. Log10/Log2/square root/cube root transformed)
- user-selected scaling option (autoscaling, Pareto scaling, range scaling, vast scaling, or level scaling)

## Outputs

- variance-scaled metabolomics data table written as a CSV file
- PCA score and loadings plots used to judge the chosen scaling option

## How to apply

Load the metabolomics table (previously normalized via reference sample/sum scaling/weight-volume and transformed via Log10, Log2, square root or cube root) as a DataFrame with pandas, as Viime's backend does. Select one of the five scaling options — autoscaling, Pareto scaling, range scaling, vast scaling, or level scaling — and scale the variance of each metabolite column using scikit-learn's scaling utilities per the Viime backend. Treat scaling as the final step of Viime's data treatment pipeline, and evaluate the choice interactively with principal component analysis (PCA) score and loadings plots to examine similarity and dissimilarity of individual groups before settling on parameters. Write the scaled data table back out as a CSV file, matching the Viime convention that processing functions accept a path to a CSV file and return a data frame.

## Related tools

- **Scikit-learn** (Provides the common statistical algorithms used by Viime's backend, including the scaling utilities applied per metabolite column for the five scaling options)
- **Pandas** (Used by Viime's backend to load and manipulate the metabolomics data table (CSV) that is scaled and written back out)
- **Viime** (The application whose final data treatment step performs scaling, offering autoscaling, Pareto, range, vast, and level scaling, evaluated via interactive PCA plots) — https://github.com/girder/viime

## Evaluation signals

- The chosen scaling option is one of the five Viime options: autoscaling, Pareto scaling, range scaling, vast scaling, or level scaling.
- Scaling is applied per metabolite column to the variance of the data, after normalization and a transformation step (Log10/Log2/square root/cube root) have already been performed.
- The output is a CSV data table with the same metabolite columns and samples as the input, but variance-scaled values.
- The scaling choice is validated via interactive PCA score and loadings plots showing expected similarity/dissimilarity of individual groups.
- The data table format matches Viime's convention of processing functions that accept a CSV path and return a data frame.

## Limitations

- Scaling is the final data treatment step, so it inherits any errors from upstream steps (e.g. missing data exceeding thresholds, non-numeric concentration data, lack of a primary ID) that were not corrected earlier.
- The README notes the OpenCPU-exposed R processing package is work-in-progress backend code for CSV processing, so backend behavior may evolve.
- Scaling alone does not expand metabolome coverage or address missingness; it addresses only the large dynamic range of metabolomics data.

## Evidence

- [abstract] Scaling options and rationale: "The options here include autoscaling, Pareto scaling, range scaling, vast scaling and level scaling."
- [abstract] Transformation step preceding scaling: "The options in this step are Log10, Log2, square root and cube root."
- [abstract] PCA used to evaluate data treatment choices: "A very important feature of the whole data treatment process is the interactive use of principal component analysis (PCA) to examine the similarity and dissimilarity of individual groups in the"
- [abstract] Scikit-learn and Pandas in Viime backend: "Data processing is done by a combination of Scikit-learn for common statistical algorithms and R packages for specialized algorithms."
- [readme] CSV-in/CSV-out convention of Viime processing functions: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
