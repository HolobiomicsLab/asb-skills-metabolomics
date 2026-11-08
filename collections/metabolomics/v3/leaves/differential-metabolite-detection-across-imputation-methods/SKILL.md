---
name: differential-metabolite-detection-across-imputation-methods
description: Decision support for whether missing-value imputation choices materially
  change the differential-metabolite lists and enriched pathways in an LC-MS untargeted
  metabolomics feature table, e.g. when a case study applied only one imputation method.
when_to_use_negative:
- The input is raw LC-MS data rather than an already-cleaned feature table — noisy-feature
  filtering (≤20% MV in QC, ≤50% MV in study groups) and peak processing must happen
  first.
- Only annotations below level 2 are available, since the subsequent analysis uses
  only level-2 annotations.
- The goal is a single definitive differential-metabolite analysis under one chosen
  imputation method rather than an inter-method sensitivity comparison.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3517
tools:
- name: tidymass
  role: Umbrella R package installing and organizing the tidyMass packages used for
    the imputation, statistical, and pathway-enrichment steps
  repo: https://gitlab.com/tidymass/tidymass
- name: masscleaner
  role: Applies each of the seven missing-value imputation methods (BPCA, svdImpute,
    missForest, zero, mean, median, minimum) to the feature table
- name: massdataset
  role: Loads the cleaned feature table as a mass_dataset object that carries prior
    processing parameters for traceability
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: massstat
  role: Runs the statistical analysis per imputed dataset to generate ranked differential-feature
    lists between study groups
- name: metpath
  role: Maps the per-method differential metabolites onto pathways and computes pathway
    enrichment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:c880446a3482aca7dfb73d84971926f5bd741712402bcf07c9ed1e4f251e00f4
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-metabolite-detection-across-imputation-methods

## Summary

A sensitivity-analysis skill that re-runs the tidyMass differential-metabolite and pathway-enrichment analysis under each missing-value imputation method available in massCleaner (BPCA, svdImpute, missForest, zero, mean, median, minimum), then compares concordance of results. It addresses the fact that the colorectal cancer case study applied only K-nearest-neighbor imputation without reporting alternatives.

## When to use

Decision support for whether missing-value imputation choices materially change the differential-metabolite lists and enriched pathways in an LC-MS untargeted metabolomics feature table, e.g. when a case study applied only one imputation method.

## When NOT to use

- The input is raw LC-MS data rather than an already-cleaned feature table — noisy-feature filtering (≤20% MV in QC, ≤50% MV in study groups) and peak processing must happen first.
- Only annotations below level 2 are available, since the subsequent analysis uses only level-2 annotations.
- The goal is a single definitive differential-metabolite analysis under one chosen imputation method rather than an inter-method sensitivity comparison.

## Inputs

- cleaned LC-MS feature table as a mass_dataset object (features pre-filtered to ≤20% MV in QC samples and ≤50% MV in study groups)
- level-2 metabolite annotations retained after redundant-annotation removal by annotation score
- study-group sample metadata (colorectal cancer case/control labels)

## Outputs

- imputation_method_comparison.csv — a collated comparison table with one complete differential-metabolite list per imputation method
- per-method pathway-enrichment results (metpath)
- overlap/concordance statistics across the seven imputation methods

## How to apply

Load the cleaned feature table as a mass_dataset object in R with massdataset; the input should already be filtered so that only features with ≤20% missing values in QC samples and ≤50% MVs in the study groups are retained. For each of the seven imputation methods implemented in massCleaner (BPCA, svdImpute, missForest, zero, mean, median, minimum), impute the missing values and run the massstat statistical analysis to produce a ranked differential-feature list between study groups. For each list, map only level-2 annotations (after redundant-annotation removal by annotation score) onto pathways with metpath and compute pathway enrichment. Collate membership and enrichment results across methods into a single comparison table with per-method lists and overlap statistics, and save it as imputation_method_comparison.csv. The rationale is that the paper's case study used only KNN imputation via massCleaner, so concordance across the seven methods quantifies how robust the reported differential metabolites and pathways are to that single choice.

## Related tools

- **tidymass** (Umbrella R package installing and organizing the tidyMass packages used for the imputation, statistical, and pathway-enrichment steps) — https://gitlab.com/tidymass/tidymass
- **masscleaner** (Applies each of the seven missing-value imputation methods (BPCA, svdImpute, missForest, zero, mean, median, minimum) to the feature table)
- **massdataset** (Loads the cleaned feature table as a mass_dataset object that carries prior processing parameters for traceability) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **massstat** (Runs the statistical analysis per imputed dataset to generate ranked differential-feature lists between study groups)
- **metpath** (Maps the per-method differential metabolites onto pathways and computes pathway enrichment)

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- imputation_method_comparison.csv contains one complete differential-metabolite list and one pathway-enrichment result for each of the seven methods — no method missing.
- Every input feature table passed the stated filter: no feature with >20% MV in QC samples or >50% MV in study groups remains.
- Only level-2 annotations appear in the pathway mapping, and redundant annotations have been removed by annotation score.
- Overlap statistics across methods are computed and reported, allowing identification of the most concordant imputation methods.
- Correctness of the overlap patterns is confirmed by expert review.

## Limitations

- The source case study reported only KNN imputation via massCleaner; no comparison against alternative imputation methods or their effect on results was provided, so baseline expectations must be generated by this analysis itself.
- Correctness of the specific statistical calls and concordance patterns requires expert review rather than automated acceptance.
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, which may constrain cross-tool validation of the per-method results.

## Evidence

- [intro] Features with more than 20% missing values in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [intro] Only level-2 annotations used: "Only the annotations with level 2 were used for subsequent analysis"
- [intro] Multiple optimization steps required: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis"
- [methods] Install tidymass from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [readme] tidymass organizes project packages: [evidence span withheld — anchor status: not_found]
