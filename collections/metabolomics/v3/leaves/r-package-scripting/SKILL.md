---
name: r-package-scripting
description: Use when the task requires reproducing or adapting a published R/Bioconductor
  package workflow by scripting its exported functions against bundled example data
  rather than reimplementing the method.
when_to_use_negative:
- Input metabolite names are already standard HMDB IDs and no pathway re-mapping is
  needed — skip the feature mapping call
- The goal is group-level pathway enrichment (MetPA, IMPaLA, MPEA) rather than individual
  patient-level pathway matrices — Lilikoi was built precisely because those tools
  cannot generate per-sample PDS matrices
- The task is batch processing outside R or in another language — this skill specifically
  covers scripting the R package's exported functions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lilikoi
  role: 'The R package being scripted: its four modules (feature mapping, dimension
    transformation, feature selection, classification/prediction) are invoked as sequential
    function calls'
  repo: https://github.com/lanagarmire/lilikoi
- name: R
  role: Scripting language/runtime in which the package functions are chained, using
    system.file() to access packaged data
- name: R/Bioconductor
  role: The ecosystem the article identifies as dominant for metabolomics scripting;
    devtools::install_github is used for installation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:d6021790754316a3adc94f29217c4ed37830e2f507237062cc6cb981af6d61ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - R/Bioconductor
  - lilikoi
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-package-scripting

## Summary

Drive a published R package end-to-end via scripted R calls — here, the Lilikoi metabolomics pipeline — chaining package functions (lilikoi.metab_to_pathway through lilikoi.adjust_model) on packaged example data loaded with system.file().

## When to use

Use when the task requires reproducing or adapting a published R/Bioconductor package workflow by scripting its exported functions against bundled example data rather than reimplementing the method.

## When NOT to use

- Input metabolite names are already standard HMDB IDs and no pathway re-mapping is needed — skip the feature mapping call
- The goal is group-level pathway enrichment (MetPA, IMPaLA, MPEA) rather than individual patient-level pathway matrices — Lilikoi was built precisely because those tools cannot generate per-sample PDS matrices
- The task is batch processing outside R or in another language — this skill specifically covers scripting the R package's exported functions

## Inputs

- R package installed from GitHub or CRAN (lilikoi)
- Bundled example CSV: plasma_breast_cancer.csv (metabolite measurements with sample Label column)
- Bundled clinical factors CSV: plasma_breast_cancer_Meta.csv (Age, Race)
- Package-shipped annotation databases (data.hmdb, data.smpdb, data.metaboAnalyst)

## Outputs

- metabolitePathwayTable: metabolite-to-HMDB-ID/pathway mapping table
- PDSmatrix: pathway deregulation score matrix (e.g., 224 pathways x 207 patients)
- significantPathways: selected pathway features (gain method, threshold 0.42)
- mlResults and finalModel: classification results and adjusted final model

## How to apply

Install the package from GitHub (devtools::install_github("lanagarmire/lilikoi")) or CRAN, then load the bundled example data with system.file('extdata', 'plasma_breast_cancer.csv', package = 'lilikoi') and read.csv(check.names = FALSE, row.names = 1), extracting metabolite names via colnames(metaboliteMeasurements)[-1]. Chain the four Lilikoi modules as sequential function calls: lilikoi.metab_to_pathway(metaboliteNames, 'name') for ID standardization, lilikoi.get_pd_scores for the pathway deregulation score (PDS) matrix, lilikoi.select_pathways (threshold = 0.42, method = 'gain') for feature selection, and lilikoi.machine_learning followed by lilikoi.adjust_model with clinical factors such as c('Age', 'Race'). Rely on the package's bundled databases (HMDB, SMPDB, MetaboAnalyst data shipped at publication date); note the README warns database updates via lilikoi.update_database() are large (>5GB) and may take >20 minutes. For faster test runs, shrink the dataset (e.g., metaboliteMeasurements[, 1:20]) or subset lilikoi::data.smpdb[1:25,], as the README demonstrates.

## Related tools

- **lilikoi** (The R package being scripted: its four modules (feature mapping, dimension transformation, feature selection, classification/prediction) are invoked as sequential function calls) — https://github.com/lanagarmire/lilikoi
- **R** (Scripting language/runtime in which the package functions are chained, using system.file() to access packaged data)
- **R/Bioconductor** (The ecosystem the article identifies as dominant for metabolomics scripting; devtools::install_github is used for installation)

## Examples

```
devtools::install_github("lanagarmire/lilikoi"); filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"); metaboliteMeasurements <- read.csv(file = filename, check.names = FALSE, row.names = 1)
```

## Evaluation signals

- The scripted pipeline runs top-to-bottom without errors on the bundled plasma_breast_cancer.csv example
- The PDS matrix has the expected shape (one row per mapped pathway, one column per sample; the paper reports 224 pathways x 207 patients for dataset 1)
- Mapping rate check: for the plasma dataset, 182 of 227 metabolites should map to standard HMDB IDs (~80%); ~20% remaining unmapped is expected
- significantPathways is non-empty after lilikoi.select_pathways with threshold = 0.42 and method = 'gain'
- finalModel is produced by lilikoi.adjust_model incorporating the requested clinical factors (e.g., Age, Race)

## Limitations

- About 20% of metabolites cannot be mapped to standard names in the databases
- Levenshtein-distance fuzzy matching is a possible cause of metabolite name mismatching
- The shipped databases reflect data as of the publication date; updating via lilikoi.update_database() is resource-heavy (>5GB, potentially >20 minutes)
- No AutoML for automatic classification algorithm selection exists yet (planned for future versions)

## Evidence

- [abstract] Lilikoi (the Hawaiian word for passion fruit) is a new and comprehensive R package for personalized pathway-based classification modeling using metabolomics data: "Lilikoi (the Hawaiian word for passion fruit) is a new and comprehensive R package for personalized pathway-based classification modeling using metabolomics data"
- [methods] metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name"): "metabolitePathwayTable <- lilikoi.metab_to_pathway(metaboliteNames, "name")"
- [methods] devtools::install_github("lanagarmire/lilikoi"): "devtools::install_github("lanagarmire/lilikoi")"
- [methods] filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"): "filename <- system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")"
- [readme] the datasets are large (>5GB) and this step may take greater than 20 minutes: "Warning: the datasets are large (>5GB) and this step may take greater than 20 minutes."
