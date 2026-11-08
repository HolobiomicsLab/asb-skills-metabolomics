---
name: metabolomics-workbench-dataset-access
description: Use when a metabolomics analysis needs a publicly available example study
  — e.g. the ST000284 three-group or ST000336 two-group targeted metabolomics datasets
  — to feed POMAShiny/POMA workflows such as group-comparison tests or covariate-adjusted
  (ANCOVA/limma) comparisons.
when_to_use_negative:
- Input is already a preprocessed feature table and target file in POMAShiny's two-CSV
  format — go straight to data upload and preprocessing panels
- The study of interest is not a targeted metabolomics dataset available on Metabolomics
  Workbench (e.g. raw MS vendor files or proteomics data without a Workbench accession)
- Proteomics or spectral-level data requiring upstream processing — the article directs
  users to RforMassSpectrometry packages for spectral data processing before migrating
  to POMA
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: POMAShiny
  role: Web application that consumes the retrieved Workbench datasets as target and
    features CSV files for preprocessing, EDA, and statistical analysis
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package underlying the analysis and host of the example dataset
    documentation used to structure the downloaded data
  repo: https://github.com/nutrimetabolomics/POMA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:a25da2b87265c2b80b2278d1ac693704f146d380ffdad5cdd5241df7bc6e1733
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metabolomics-workbench-dataset-access

## Summary

Retrieves the publicly available example metabolomics datasets ST000284 and ST000336 from Metabolomics Workbench and reshapes them into the two-CSV target (metadata) and features format required by POMAShiny/POMA, enabling reproducible demonstration and benchmarking of downstream univariate, covariate-adjusted, and multivariate analyses.

## When to use

Use when a metabolomics analysis needs a publicly available example study — e.g. the ST000284 three-group or ST000336 two-group targeted metabolomics datasets — to feed POMAShiny/POMA workflows such as group-comparison tests or covariate-adjusted (ANCOVA/limma) comparisons.

## When NOT to use

- Input is already a preprocessed feature table and target file in POMAShiny's two-CSV format — go straight to data upload and preprocessing panels
- The study of interest is not a targeted metabolomics dataset available on Metabolomics Workbench (e.g. raw MS vendor files or proteomics data without a Workbench accession)
- Proteomics or spectral-level data requiring upstream processing — the article directs users to RforMassSpectrometry packages for spectral data processing before migrating to POMA

## Inputs

- Metabolomics Workbench study identifier (e.g. ST000284 or ST000336)
- Study documentation from github.com/nutrimetabolomics/POMA
- Target (metadata) CSV file with sample group labels and covariates
- Features CSV file with metabolite intensity values per sample

## Outputs

- Loaded feature table and target metadata ready for POMAShiny/POMA upload
- Group-labeled dataset supporting three-group (ST000284) or two-group (ST000336) univariate comparisons

## How to apply

Fetch the study of interest from Metabolomics Workbench (both example datasets are freely available at https://www.metabolomicsworkbench.org), noting that ST000284 is a targeted metabolomics three-group study and ST000336 is a two-group study. Convert or download the data into the two comma-separated CSV files POMAShiny requires: the target (metadata) file with sample group labels and any study covariates, and the features file with metabolite intensities per sample. Dataset documentation (study design and file structure) is available at https://github.com/nutrimetabolomics/POMA, which hosts both data set documentations. Once loaded into R, the target file's group column drives plain group-comparison tests (t-test/ANOVA) via POMA, and additional covariate columns enable covariate-adjusted ANCOVA/limma models — the paper claims such covariate-combining methods improve the accuracy and understanding of results. Verify group structure (three groups for ST000284, two for ST000336) before proceeding to preprocessing panels such as missing-value imputation (20% per-group default removal threshold) and normalization.

## Related tools

- **POMAShiny** (Web application that consumes the retrieved Workbench datasets as target and features CSV files for preprocessing, EDA, and statistical analysis) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package underlying the analysis and host of the example dataset documentation used to structure the downloaded data) — https://github.com/nutrimetabolomics/POMA

## Examples

```
Rscript -e 'features <- read.csv("ST000284_features.csv"); target <- read.csv("ST000284_target.csv")'
```

## Evaluation signals

- Retrieved dataset splits into exactly two CSV files (target/metadata and features) matching POMAShiny's required input structure
- ST000284 loads with its documented three-group study structure; ST000336 with two groups
- Sample identifiers in the target file match those in the features file row/column keys
- Dataset documentation at github.com/nutrimetabolomics/POMA matches the downloaded study's design and variables
- Group labels and covariate columns in the target file support both plain ANOVA/t-test and covariate-adjusted ANCOVA/limma model fitting

## Limitations

- The paper provides no software version numbers for POMAShiny or POMA and no changelog, so dataset-format expectations may drift across releases
- No benchmarking or quantitative evaluation of POMAShiny against other tools is reported, so retrieved datasets cannot be validated against an external reference analysis
- Only two example datasets (ST000284, ST000336) are documented; other Workbench studies may require manual restructuring into the two-CSV format

## Evidence

- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [results] the example data set ST000336 consists of a targeted metabolomics two-group study: "the example data set ST000336 consists of a targeted metabolomics two-group study"
- [results] both freely available at https://www.metabolomicsworkbench.org: "both freely available at https://www.metabolomicsworkbench.org"
- [results] Both data set documentations are available at https://github.com/nutrimetabolomics/POMA.: "Both data set documentations are available at https://github.com/nutrimetabolomics/POMA."
- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
