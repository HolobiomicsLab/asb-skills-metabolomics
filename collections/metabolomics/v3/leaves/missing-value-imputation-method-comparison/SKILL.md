---
name: missing-value-imputation-method-comparison
description: Use when a cleaned LC-MS feature table (mass_dataset object) still contains
  missing values and you need to assess whether the choice of imputation method changes
  the differential-metabolite and pathway-enrichment conclusions between study groups.
when_to_use_negative:
- The feature table has no missing values to impute, or is a raw peak table that has
  not yet passed the noisy-feature removal step (>20% MVs in QC or >50% MVs in study
  groups).
- You only need a single imputation method for a routine analysis with no sensitivity
  assessment — the case study used K-nearest neighbors alone for that purpose.
- The input is raw LC-MS data requiring peak picking and grouping (e.g. mzML processing
  with MZmine/MS-DIAL/XCMS), not a processed feature table.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: tidyMass
  role: Overall R-based computational framework whose modular architecture hosts the
    imputation comparison workflow
  repo: https://github.com/tidymass/tidymass
- name: masscleaner
  role: Provides the seven MV imputation methods (BPCA, svdImpute, missForest, zero,
    mean, median, minimum) applied to the feature table
- name: massdataset
  role: Loads and manages the cleaned feature table as a mass_dataset object for traceable
    processing
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: massstat
  role: Runs the differential-expression-style statistical analysis on each imputed
    dataset to generate ranked differential-feature lists
- name: metpath
  role: Maps level-2 annotated metabolites onto metabolic pathways and computes pathway
    enrichment per imputation method
- name: metid
  role: Supplies the metabolite annotations used for pathway mapping in the comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:c80cc1891c785d06813d9c7abca86d4fad122c6ff50b610ccda757b6bbbbb668
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - dplyr
  - ggplot2
  - masscleaner
  - massdataset
  - massstat
  - metid
  - metpath
  - purrr
  - tidyMass
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# missing-value-imputation-method-comparison

## Summary

Systematically compare how alternative missing-value (MV) imputation methods available in tidyMass's massCleaner (BPCA, svdImpute, missForest, zero, mean, median, minimum) affect downstream differentially expressed metabolites and enriched pathways in an LC-MS untargeted metabolomics study.

## When to use

Use when a cleaned LC-MS feature table (mass_dataset object) still contains missing values and you need to assess whether the choice of imputation method changes the differential-metabolite and pathway-enrichment conclusions between study groups.

## When NOT to use

- The feature table has no missing values to impute, or is a raw peak table that has not yet passed the noisy-feature removal step (>20% MVs in QC or >50% MVs in study groups).
- You only need a single imputation method for a routine analysis with no sensitivity assessment — the case study used K-nearest neighbors alone for that purpose.
- The input is raw LC-MS data requiring peak picking and grouping (e.g. mzML processing with MZmine/MS-DIAL/XCMS), not a processed feature table.

## Inputs

- Cleaned feature table as a mass_dataset object (features already filtered to <=20% MVs in QC samples and <=50% MVs in study groups)
- Study-group sample metadata associated with the mass_dataset object
- Level-2 metabolite annotations retained after redundant-annotation removal

## Outputs

- imputation_method_comparison.csv containing per-method differential-metabolite lists and pathway-enrichment results with overlap statistics
- Per-method ranked differential-feature lists from massstat
- Per-method pathway-enrichment results from metpath

## How to apply

Load the cleaned feature table — already filtered so features with more than 20% missing values in QC samples or more than 50% MVs in all the study groups were removed — as a mass_dataset object with massdataset. Apply each massCleaner imputation method in turn (BPCA, svdImpute, missForest, zero, mean, median, minimum) to the MVs in the feature table. For each imputed dataset, run the statistical analysis in massstat to generate a ranked list of differential features between the study groups, then map only level-2 annotated metabolites (after redundant-annotation removal based on annotation score) onto metabolic pathways and compute enrichment with metpath. Collate per-method differential-metabolite membership and pathway-enrichment results into a single comparison table with overlap statistics, saved as imputation_method_comparison.csv. The rationale is that the tidyMass case study applied only K-nearest-neighbors imputation via massCleaner without reporting sensitivity to this choice, so benchmarking all seven methods tests the robustness of the biological conclusions to this preprocessing decision.

## Related tools

- **tidyMass** (Overall R-based computational framework whose modular architecture hosts the imputation comparison workflow) — https://github.com/tidymass/tidymass
- **masscleaner** (Provides the seven MV imputation methods (BPCA, svdImpute, missForest, zero, mean, median, minimum) applied to the feature table)
- **massdataset** (Loads and manages the cleaned feature table as a mass_dataset object for traceable processing) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **massstat** (Runs the differential-expression-style statistical analysis on each imputed dataset to generate ranked differential-feature lists)
- **metpath** (Maps level-2 annotated metabolites onto metabolic pathways and computes pathway enrichment per imputation method)
- **metid** (Supplies the metabolite annotations used for pathway mapping in the comparison)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- The comparison table contains one complete differential-metabolite list and one pathway-enrichment result for each of the seven imputation methods, with no method missing.
- Each per-method differential list is generated from the same cleaned mass_dataset input, so only the imputation step differs between arms.
- Overlap statistics between methods are computed and recorded, enabling expert review of which methods yield concordant results.
- Only level-2 annotations (after redundant-annotation removal by annotation score) feed the metpath pathway mapping.
- The output file imputation_method_comparison.csv is produced and covers all seven methods.

## Limitations

- The original tidyMass case study used only K-nearest neighbors imputation via massCleaner and did not report comparison against alternative methods, so no published sensitivity baseline exists for this case study.
- The imputation comparison is only valid downstream of the noisy-feature filter (features with more than 20% MVs in QC samples or more than 50% MVs in all study groups removed); it cannot recover features already removed.
- The correctness of specific statistical calls in the cross-method comparison is confirmed by expert review of overlap patterns, not by an automated ground truth.

## Evidence

- [intro] MV imputation thresholds: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [intro] multi-step optimization rationale: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment"
- [abstract] uniform data format for traceability: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [readme] tidymass package organization: [evidence span withheld — anchor status: not_found]
