---
name: feature-outlier-detection
description: Use when raw ion-concentration data (e.g. ICP-MS ppm values for Ca44,
  Cd111, Fe56, K39...) contain extreme readings or unreliable samples that must be
  flagged before batch correction, standardisation and symbolisation.
when_to_use_negative:
- The data are already normalised and standardised — IonFlow can consume such data
  directly for network extraction, so re-running raw-data outlier flagging would double-process
  them
- The goal is batch correction or symbolic-profile extraction itself rather than screening
  out unreliable samples before those steps
- The input is not element-concentration ionomics data (e.g. already-derived z-score
  tables), where IQR fences on raw ppm values would be meaningless
edam_topics:
- http://edamontology.org/topic_3389
tools:
- name: IonFlow
  role: R package whose PreProcessing function implements per-ion outlier detection
    (IQR outer fences, MAD, or log.FC.distance with thres_outl)
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web interface exposing the IonFlow pre-processing/outlier-removal step to
    non-command-line users
  repo: https://github.com/galaxyproject/galaxy
- name: MetaboFlow
  role: Upstream repository hosting the IonFlow R package (installed via devtools::install_github
    from subdir='IonFlow') containing the PreProcessing implementation
  repo: https://github.com/AlinaPeluso/MetaboFlow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:214c52ae115201abe2c3ef59cd6cb479fa0dc31fc97be933496a95b10505f79d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - MetaboFlow
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3389
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# feature-outlier-detection

## Summary

Flags extreme per-ion outliers in raw ICP-MS ionomics data (e.g. S. cerevisiae knockout concentrations) as part of IonFlow's PreProcessing stage, so that unreliable samples do not propagate into z-score and symbolic profiles.

## When to use

Use when raw ion-concentration data (e.g. ICP-MS ppm values for Ca44, Cd111, Fe56, K39...) contain extreme readings or unreliable samples that must be flagged before batch correction, standardisation and symbolisation.

## When NOT to use

- The data are already normalised and standardised — IonFlow can consume such data directly for network extraction, so re-running raw-data outlier flagging would double-process them
- The goal is batch correction or symbolic-profile extraction itself rather than screening out unreliable samples before those steps
- The input is not element-concentration ionomics data (e.g. already-derived z-score tables), where IQR fences on raw ppm values would be meaningless

## Inputs

- Raw data frame of ion concentrations (e.g. IonData: ICP-MS ppm values for 14 ions measured in 1454 single-gene haploid S. cerevisiae knockouts, 2/4/8 replicas per mutant, YDL227C as control strain)
- Optional pre-defined ion standard deviations (stdev) computed from control samples

## Outputs

- pre_proc$stats.outliers table (per-ion counts of not_outlier, outlier, outlier(%) )
- Cleaned dataset with extreme outliers removed, feeding downstream standardisation and data.line.zscores / data.line.symb outputs
- Diagnostic plots of outlier detection

## How to apply

Run the PreProcessing function of the IonFlow R package on the raw data frame of ion concentrations. Within it, choose an outlier method — IQR fences, MAD-based, or log.FC.distance — controlled by thres_outl. The IQR default defines a lower outer fence Q1 - 3*IQ and an upper outer fence Q3 + 3*IQ per ion (Q1 and Q3 being the first and third quantiles); any point beyond the outer fence is an extreme outlier. Detection is performed after log-transformation and median batch correction (scaling to each ion's within-batch median), because raw concentrations span several orders of magnitude (e.g. K up to 17777 ppm, Co below 0.01 ppm) and batch effects dominate raw variability. Counts of flagged points per ion are reported in pre_proc$stats.outliers for sanity-checking before the cleaned data feed standardisation (std/mad/custom) and symbolisation at thres_symb.

## Related tools

- **IonFlow** (R package whose PreProcessing function implements per-ion outlier detection (IQR outer fences, MAD, or log.FC.distance with thres_outl)) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Web interface exposing the IonFlow pre-processing/outlier-removal step to non-command-line users) — https://github.com/galaxyproject/galaxy
- **MetaboFlow** (Upstream repository hosting the IonFlow R package (installed via devtools::install_github from subdir='IonFlow') containing the PreProcessing implementation) — https://github.com/AlinaPeluso/MetaboFlow

## Examples

```
pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd); pre_proc$stats.outliers
```

## Evaluation signals

- pre_proc$stats.outliers reports both not_outlier and outlier counts for every ion, with outlier(%) in the expected low range (README benchmark: 0.01% for Mg/P/K up to 0.22% for Ca)
- Flagged points lie strictly beyond Q1 - 3*IQ or Q3 + 3*IQ of the per-ion (log-transformed, median-batch-corrected) distribution
- After median batch correction, the median of each ion's corrected distribution equals 0 (README tables show Median = 0 for all 14 ions)
- The cleaned dataset has no missing values and feeds forward to produce non-empty data.line.zscores and data.line.symb tables
- stats.raw_data vs stats.median_batch_corrected_data show compressed min/max ranges (e.g. Fe max drops from 6624.5 to 1.869), confirming outlier influence is contained before standardisation

## Limitations

- The choice of outlier method (log.FC.distance, mad, IQR) and thres_outl changes which samples are dropped; the README's IQR-outer-fence benchmark removes only 0.01–0.22% per ion, so heavily contaminated batches may need a stricter threshold
- Raw ion variability is very high across knockouts and within batches (e.g. Fe variance 5154.6), so outlier calls are ion-specific and sensitive to the log-transform/batch-correction order
- The paper reports no quantitative benchmark results or performance metrics for the case studies, so correctness must be judged from the stats tables and diagnostic plots rather than external accuracy figures

## Evidence

- [readme] IQR outer-fence rule per ion: [evidence span withheld — anchor status: external_doc]
- [readme] Definition of extreme outlier: [evidence span withheld — anchor status: external_doc]
- [readme] Rationale for pre-processing/outlier removal: [evidence span withheld — anchor status: external_doc]
- [readme] Outlier detection follows log-transform and median batch correction: [evidence span withheld — anchor status: external_doc]
- [abstract] Pipeline context for raw-data processing: "designed as a pipeline that can process raw data to enable exploration and interpretation"
