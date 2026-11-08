---
name: r-package-demo-dataset-handling
description: Use when a Lilikoi v2.0 analysis run needs a known-good metabolomics
  input and paired phenotype metadata, e.g. to demo or smoke-test PCA/t-SNE/SOV, feature
  selection, or classification modules before applying them to user data.
when_to_use_negative:
- Your metabolomics data is already normalized/imputed and formatted as a Lilikoi-expected
  dataSet + Metadata pair — skip the demo fixture and load your own data directly
- You need raw-format conversion (e.g. mzML/NetCDF preprocessing) — Lilikoi v2.0 starts
  from a processed metabolite-by-sample table, not instrument raw files
- You are running non-metabolomics omics data through the package — the demo dataset
  and Label-based metadata structure are metabolomics-specific
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Lilikoi v2.0
  role: R package whose lilikoi.Loaddata function loads the built-in plasma_breast_cancer.csv
    demo dataset and returns the Metadata/dataSet pair consumed by its exploratory
    (PCA, t-SNE, SOV), pathway, and classification modules
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the lilikoi package and its demo dataset
    are loaded and the workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:27d16a5604346c08c42ea159f9dbc4805f597e1f8418b956b85d8dfa43d14192
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# r-package-demo-dataset-handling

## Summary

Loads and prepares the Lilikoi v2.0 built-in demo metabolomics dataset (plasma_breast_cancer.csv) via lilikoi.Loaddata, splitting it into a dataSet matrix and a Metadata table so downstream modules (exploratory PCA/t-SNE/SOV, pathway analysis, ML) have correctly paired inputs.

## When to use

Use when a Lilikoi v2.0 analysis run needs a known-good metabolomics input and paired phenotype metadata, e.g. to demo or smoke-test PCA/t-SNE/SOV, feature selection, or classification modules before applying them to user data.

## When NOT to use

- Your metabolomics data is already normalized/imputed and formatted as a Lilikoi-expected dataSet + Metadata pair — skip the demo fixture and load your own data directly
- You need raw-format conversion (e.g. mzML/NetCDF preprocessing) — Lilikoi v2.0 starts from a processed metabolite-by-sample table, not instrument raw files
- You are running non-metabolomics omics data through the package — the demo dataset and Label-based metadata structure are metabolomics-specific

## Inputs

- Built-in package dataset plasma_breast_cancer.csv (under inst/extdata of the lilikoi R package)
- Optional user-supplied metabolomics abundance matrix plus a paired Metadata table with a phenotype Label column (as an alternative to the demo fixture)

## Outputs

- dt$Metadata: sample/phenotype metadata table (includes Label) used for SOV confounder assessment
- dt$dataSet: metabolite abundance data frame aligned to Metadata samples
- Downstream artifacts when fed to lilikoi.explr: PCA and t-SNE plot image files and an SOV table of ANOVA F-scores per metadata variable

## How to apply

Call lilikoi.Loaddata pointing at the package-internal file system.file('extdata', 'plasma_breast_cancer.csv', package='lilikoi'); the function returns a list from which you extract dt$Metadata (sample/phenotype table including a Label column) and dt$dataSet (metabolite abundance matrix). Keep Metadata and dataSet row-aligned, since the exploratory module lilikoi.explr consumes the metabolites data frame together with the demographic/metadata data frame to compute ANOVA F-scores for source-of-variation (SOV) and, when pca=TRUE and tsne=TRUE, renders PCA and t-SNE plots via the M3C visualization routines. Using the built-in demo data guarantees the column names, sample labels, and metabolite identifiers match what the Lilikoi modules expect, which makes it the right fixture for validating a fresh install (installed via install.packages('lilikoi') or devtools::install_github('lanagarmire/lilikoi2')) before adapting the workflow to external metabolomics data.

## Related tools

- **Lilikoi v2.0** (R package whose lilikoi.Loaddata function loads the built-in plasma_breast_cancer.csv demo dataset and returns the Metadata/dataSet pair consumed by its exploratory (PCA, t-SNE, SOV), pathway, and classification modules) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the lilikoi package and its demo dataset are loaded and the workflow is executed)

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")); Metadata <- dt$Metadata; dataSet <- dt$dataSet
```

## Evaluation signals

- lilikoi.Loaddata returns a list with both Metadata and dataSet components, and nrow(dataSet) matches the number of samples described in Metadata (row-aligned pairing)
- Metadata contains a Label column usable as measurementLabels for downstream lilikoi.machine_learning calls
- When the loaded pair is passed to lilikoi.explr with pca=TRUE and tsne=TRUE, the expected PCA/t-SNE plot image files are written to disk
- The SOV result table contains ANOVA F-score entries for each tested metadata variable (expert review of table completeness)
- system.file('extdata', 'plasma_breast_cancer.csv', package='lilikoi') resolves to a non-empty path, confirming the package (not just the script) is installed

## Limitations

- The demo fixture only validates the package-expected schema (metabolite abundance matrix + Label-bearing Metadata); real user datasets may need prior normalization/imputation, which Lilikoi v2.0 provides as separate preprocessing methods
- Example code in the README references file='system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")', so paths break if the package is not installed locally (CRAN or devtools::install_github('lanagarmire/lilikoi2'))
- Detailed parameter-level documentation is limited in the surfaced article sections; the user guide (Lilikoi2 User Guide.Rmd in the repo) is the referenced source for more examples

## Evidence

- [other] Load the built-in plasma_breast_cancer.csv metabolomics dataset and its demo metadata table into the R environment.: [evidence span withheld — anchor status: not_found]
- [other] The lilikoi.explr function performs source-of-variation analysis using user-provided metabolites and demographic data frames: [evidence span withheld — anchor status: not_found]
- [intro] we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis (e.g., principal component analysis: "we also include additional preprocessing methods for metabolomics data analysis (e.g., normalization, imputation) and tools for exploratory data analysis (e.g., principal component analysis"
- [readme] dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")): "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))
Metadata <- dt$Metadata
dataSet <- dt$dataSet"
- [readme] Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression.: "Additionally, Lilikoi v2 supports data preprocessing, exploratory analysis, pathway visualization and metabolite-pathway regression."
