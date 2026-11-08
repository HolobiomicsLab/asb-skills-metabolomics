---
name: lc-ms-feature-table-quality-filtering
description: Use when a processed LC–MS peak/feature table still contains missing
  values and you must remove noisy features by MV thresholds before imputation, differential
  analysis, and pathway enrichment in the tidyMass framework.
when_to_use_negative:
- Input is raw instrument data (e.g., mzML) that still requires peak picking and grouping
  — that upstream raw-data-processing stage must run first before any feature-table
  MV filtering
- The feature table has already been missing-value filtered and imputed, and the goal
  is only downstream statistics or visualization
- You need to design a new imputation algorithm rather than compare the existing massCleaner
  methods against each other
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massdataset
  role: Provides the mass_dataset object format used to load and trace the feature
    table before filtering
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: masscleaner
  role: Performs the noisy-feature MV filtering and applies each imputation method
    (KNN, BPCA, svdImpute, missForest, zero, mean, median, minimum)
  repo: https://www.tidymass.org/
- name: massstat
  role: Runs the differential-metabolite statistical analysis on each imputed dataset
  repo: https://www.tidymass.org/
- name: metpath
  role: Computes pathway enrichment from the per-method differential-metabolite lists
  repo: https://www.tidymass.org/
- name: tidymass
  role: Umbrella package installing the tidyMass ecosystem (masscleaner, massdataset,
    massstat, metpath) used for this workflow
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:6be03b641f8e1f715fccbf3c69252d4b250be927430b0b4580f8c80ee9f1e4ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dplyr
  - ggplot2
  - masscleaner
  - massdataset
  - massstat
  - metpath
  - purrr
  - tidymass
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lc-ms-feature-table-quality-filtering

## Summary

Filters noisy features from an LC–MS untargeted metabolomics feature table by missing-value fraction — removing features with >20% MVs in QC samples or >50% MVs in study groups — and, per the tidyMass case-study card, evaluates downstream sensitivity by repeating imputation (KNN vs BPCA, svdImpute, missForest, zero, mean, median, minimum) and differential/pathway analysis across methods.

## When to use

Use when a processed LC–MS peak/feature table still contains missing values and you must remove noisy features by MV thresholds before imputation, differential analysis, and pathway enrichment in the tidyMass framework.

## When NOT to use

- Input is raw instrument data (e.g., mzML) that still requires peak picking and grouping — that upstream raw-data-processing stage must run first before any feature-table MV filtering
- The feature table has already been missing-value filtered and imputed, and the goal is only downstream statistics or visualization
- You need to design a new imputation algorithm rather than compare the existing massCleaner methods against each other

## Inputs

- Processed LC–MS feature (peak) intensity table with missing values, containing QC samples and study-group samples (e.g., colorectal cancer case study table)
- Sample metadata distinguishing QC samples from study groups (for the 20%/50% MV thresholding)
- mass_dataset object (tidyMass uniform data format) holding the feature table and processing parameters
- Level-2 metabolite annotations retained after redundant-annotation removal

## Outputs

- Filtered feature table retaining only features with ≤20% MVs in QC samples and ≤50% MVs in study groups
- Per-method imputed feature tables (KNN, BPCA, svdImpute, missForest, zero, mean, median, minimum)
- Per-method differential-metabolite ranked lists (massstat) and pathway-enrichment results (metpath)
- imputation_method_comparison.csv with per-method lists and overlap statistics

## How to apply

Load the cleaned feature table as a mass_dataset object in R with massdataset, so prior processing steps and parameters remain traceable. Compute per-feature missing-value fractions separately in QC samples and in study groups, and remove any feature with more than 20% MVs in QC samples or more than 50% MVs in all the study groups, as these are considered noisy features in the tidyMass case study. Impute the remaining MVs with massCleaner — the case study used K-nearest neighbors by default, but to test imputation sensitivity apply each massCleaner method (BPCA, svdImpute, missForest, zero, mean, median, minimum) to the same filtered table. For each imputed dataset run the differential-metabolite statistical analysis in massstat, then map level-2 metabolite annotations (after redundant-annotation removal by annotation score) to pathways with metpath. Collate per-method differential-metabolite lists and enrichment results, with overlap statistics, into a single comparison table saved as imputation_method_comparison.csv. The rationale is that imputation choice is an unexamined pipeline knob: the case study applied only KNN and never compared alternatives, so re-running the branch across methods quantifies how robust the reported differential metabolites and pathways are to the MV-filter/imputation step.

## Related tools

- **massdataset** (Provides the mass_dataset object format used to load and trace the feature table before filtering) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **masscleaner** (Performs the noisy-feature MV filtering and applies each imputation method (KNN, BPCA, svdImpute, missForest, zero, mean, median, minimum)) — https://www.tidymass.org/
- **massstat** (Runs the differential-metabolite statistical analysis on each imputed dataset) — https://www.tidymass.org/
- **metpath** (Computes pathway enrichment from the per-method differential-metabolite lists) — https://www.tidymass.org/
- **tidymass** (Umbrella package installing the tidyMass ecosystem (masscleaner, massdataset, massstat, metpath) used for this workflow) — https://github.com/tidymass/tidymass

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- After filtering, no retained feature exceeds 20% MVs in QC samples or 50% MVs in any study group (recompute MV fractions per feature to verify)
- imputation_method_comparison.csv contains one complete differential-metabolite list and one pathway-enrichment result for each of the seven massCleaner imputation methods, with no method missing
- Per-method overlap statistics are recorded and cross-method overlap patterns are reviewed by an expert to confirm correctness of the statistical calls
- Only level-2 (or better) annotations are mapped to pathways, and redundant annotations have been removed based on the annotation score
- The mass_dataset object retains processing parameters so prior analysis steps and parameters are traceable

## Limitations

- The published case study used only KNN imputation and did not report any comparison against alternative imputation methods, so the multi-method comparison here is an extension rather than a reproduction of the paper's results
- MV thresholds (20% in QC, 50% in study groups) come from the tidyMass case study and are not validated as universally optimal; alternative cutoffs may change which features are deemed noisy
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though mass_dataset can be converted to other tools' data structures

## Evidence

- [intro] Noisy-feature MV thresholds: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [intro] Only level-2 annotations used downstream: "Only the annotations with level 2 were used for subsequent analysis"
- [abstract] Uniform traceable data format: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [intro] Docker environment for the case study: "A docker image containing the data, code, and analysis environment is also provided"
- [methods] tidyMass installation from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
