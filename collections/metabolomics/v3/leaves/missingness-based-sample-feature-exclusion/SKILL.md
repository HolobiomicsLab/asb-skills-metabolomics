---
name: missingness-based-sample-feature-exclusion
description: Use when raw un/targeted metabolomics sample-by-feature matrices (e.g.
  Metabolon or Nightingale exports) need preanalysis cleaning by dropping samples
  and metabolites above missingness thresholds before downstream analysis.
when_to_use_negative:
- Data has already passed missingness QC or is a curated feature table — re-running
  would drop nothing or re-filter inappropriately
- The goal is exploratory visualization or report generation rather than deciding
  which samples/metabolites to exclude
- Non-metabolomics omics data (e.g. gene expression) where metaboprep's missingness
  and peak-area QC semantics do not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: metaboprep
  role: Implements the two-stage (extreme + user-defined) missingness filtering of
    samples and features within quality_control() and records exclusion reason codes
    in the Metaboprep object
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep package and its quality_control/summary
    functions are executed
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:268e0c2d2f74abdcce354c987fdd2cdf84a350592f30c7a7f4a13f6791773fef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# missingness-based-sample-feature-exclusion

## Summary

A quality-control filtering step from the metaboprep R package that excludes samples and metabolites (features) whose missingness exceeds user-defined thresholds (e.g. 0.2 at the user-defined level, >=80% at the extreme level), yielding a high-quality preanalysis dataset for metabolomics platforms such as Metabolon UPLC-MS/MS and Nightingale NMR.

## When to use

Use when raw un/targeted metabolomics sample-by-feature matrices (e.g. Metabolon or Nightingale exports) need preanalysis cleaning by dropping samples and metabolites above missingness thresholds before downstream analysis.

## When NOT to use

- Data has already passed missingness QC or is a curated feature table — re-running would drop nothing or re-filter inappropriately
- The goal is exploratory visualization or report generation rather than deciding which samples/metabolites to exclude
- Non-metabolomics omics data (e.g. gene expression) where metaboprep's missingness and peak-area QC semantics do not apply

## Inputs

- Platform metabolomics export (e.g. Metabolon .xlsx worksheet such as OrigScale, Nightingale .xlsx, Olink .txt, SomaLogic .adat) with sample and feature annotation sheets
- User-defined QC thresholds (sample_missingness, feature_missingness, e.g. 0.2 each)
- R Metaboprep object created via read_metabolon(return_Metaboprep=TRUE) or Metaboprep(data, features, samples)

## Outputs

- Filtered sample-by-metabolite matrix in the 'qc' data layer (exportable as tab-delimited text, e.g. qc_filtered_dataset.txt)
- Exclusion counts by reason code (e.g. user_defined_sample_missingness, extreme_feature_missingness) from summary(), exportable as qc_exclusion_summary.txt
- Tab-delimited summary files and an HTML QC report

## How to apply

Load the platform export with read_metabolon (return_Metaboprep=TRUE) or the equivalent reader, wrap it in a Metaboprep object, and call quality_control(source_layer='input') with user-defined thresholds. Missingness filtering proceeds in two stages: an extreme-missingness screen (>=80% missing excludes samples/features unconditionally) followed by a user-defined threshold (e.g. sample_missingness=0.2, feature_missingness=0.2) applied after extreme screens. Samples flagged as missingness outliers are excluded first, then features exceeding the feature_missingness cutoff are dropped (with feature_selection='least_missingness' available to prioritize retention); the qc data layer stores exclusions and reason codes. Applied to BiB_MS-1 and ALSPAC_F24 with these thresholds, the pipeline excluded 15 and 3 samples respectively, no ALSPAC_F24 metabolites, and 24% of BiB_MS-1 metabolites via missingness criteria. Inspect exclusions via summary() on the Metaboprep object, which reports per-code counts such as 'user_defined_sample_missingness' and 'user_defined_feature_missingness'.

## Related tools

- **metaboprep** (Implements the two-stage (extreme + user-defined) missingness filtering of samples and features within quality_control() and records exclusion reason codes in the Metaboprep object) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep package and its quality_control/summary functions are executed) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment="leave_be", winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- summary() on the QC'd Metaboprep object reports nonzero counts under 'user_defined_sample_missingness' and 'user_defined_feature_missingness' consistent with the chosen 0.2 thresholds
- Reproducing the paper's BiB_MS-1 / ALSPAC_F24 run yields 15 and 3 excluded samples, 24% of BiB_MS-1 metabolites excluded by missingness, and 0 ALSPAC_F24 metabolites excluded
- No samples or features remain in the qc layer with missingness >=80% (extreme screens report 0 exclusions when pre-screened)
- Excluded entries carry populated 'excluded' and 'reason_excluded' columns in sample and feature annotation metadata
- Output dataset is written as tab-delimited text and row/column counts equal input counts minus the reported exclusion totals

## Limitations

- Thresholds are user-defined and package is marked experimental, so exact exclusion counts depend on the chosen cutoffs (e.g. 0.2 here vs. other defaults)
- metaboprep notes 'there is currently a lack of standardization and reporting transparency' in metabolomics preprocessing generally; thresholds must be reported for reproducibility
- Features failing the missingness cutoff are permanently dropped from the qc layer, so borderline metabolites cannot be recovered without re-running QC with relaxed thresholds

## Evidence

- [intro] Perform data filtering with standard pipeline and user-defined thresholds: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [methods] sample_missingness threshold in methods: "sample_missingness  = 0.2"
- [methods] feature_missingness threshold in methods: "feature_missingness = 0.2"
- [readme] README quality_control call with thresholds: "quality_control( source_layer        = "input", 
                                     sample_missingness  = 0.2, 
                                     feature_missingness = 0.2,"
- [readme] README summary output showing missingness exclusion counts: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract quality-metric selection rationale: "enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
