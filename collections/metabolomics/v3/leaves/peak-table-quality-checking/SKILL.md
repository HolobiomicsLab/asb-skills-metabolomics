---
name: peak-table-quality-checking
description: Use when you have an integrated, time-aligned, imputed peak table from
  untargeted LC-MS metabolomics and need to filter peaks for quality checking and
  account for technical variation prior to downstream grouping or statistics.
when_to_use_negative:
- Input is a raw vendor file (e.g. .CDF) or MS2 mzXML spectra rather than an imputed
  peak table — run integration and imputation first
- Data have not yet been processed for missing values and artifacts; filtering on
  a peak table with remaining missing values will misjudge feature quality
- You need annotation or tentative identification by database search — that is the
  separate OUKS '5. Annotation.R' step, not quality filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OUKS
  role: Supplies the '6. Filtering.R' script that implements peaks filtering for quality
    checking and accounting of technical variation, including D-Ratio filtering
  repo: https://github.com/plyush1993/OUKS
- name: R (>= 4.1.2)
  role: Runtime for executing the OUKS filtering script
  repo: https://cloud.r-project.org/
- name: RStudio
  role: Recommended IDE for running OUKS scripts, which are adapted to the RStudio
    headings system
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.1c00392
    title: '<i>Omics Untargeted Key Script</i>: R-Based Software Toolbox for Untargeted
      Metabolomics with Bladder Cancer Biomarkers Discovery Case Study'
  source_content_hash: sha256:739896ce1bba7e25c3a7f0626240358ab33e97fbd5103f00de1ba0b474a1a9d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/plyush1993/OUKS
  tools:
  - MAI
  - OUKS
  - R (>= 4.1.2)
  - RStudio
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# peak-table-quality-checking

## Summary

Quality-checking and filtering of an LC-MS untargeted metabolomics peak table using the OUKS '6. Filtering.R' script, which removes poor-quality features and accounts for technical variation before statistical analysis.

## When to use

Use when you have an integrated, time-aligned, imputed peak table from untargeted LC-MS metabolomics and need to filter peaks for quality checking and account for technical variation prior to downstream grouping or statistics.

## When NOT to use

- Input is a raw vendor file (e.g. .CDF) or MS2 mzXML spectra rather than an imputed peak table — run integration and imputation first
- Data have not yet been processed for missing values and artifacts; filtering on a peak table with remaining missing values will misjudge feature quality
- You need annotation or tentative identification by database search — that is the separate OUKS '5. Annotation.R' step, not quality filtering

## Inputs

- Imputed, artifact-removed peak table in .csv format (output of OUKS '3. Imputation.R', Datasets (csv) folder format)
- Sample metadata table (metadata.csv with sample class labels including QC samples)
- Run order / batch information for technical-variation accounting

## Outputs

- Filtered peak table (.csv) with low-quality and D-Ratio-failing features removed
- Retained feature matrix suitable for downstream normalization, grouping, and statistics

## How to apply

Feed the peak matrix produced by the upstream OUKS imputation step into '6. Filtering.R', which performs peaks filtering for quality checking and accounting of technical variation. The script applies quality-based cutoffs, including D-Ratio filtering added in later OUKS versions, so that features whose biological signal is not distinguishable from technical variation are dropped. Run it after '3. Imputation.R' and before '7. Normalization.R' so that downstream normalization, grouping, and statistics operate only on retained features. Output should be a filtered peak table written to the Datasets (csv) folder in the OUKS .csv format. Because the toolbox is Windows-friendly and version-sensitive, keep R ≥ 4.1.2 and match package versions stored in the 'Required packages' archive when reproducing the filtering behavior.

## Related tools

- **OUKS** (Supplies the '6. Filtering.R' script that implements peaks filtering for quality checking and accounting of technical variation, including D-Ratio filtering) — https://github.com/plyush1993/OUKS
- **R (>= 4.1.2)** (Runtime for executing the OUKS filtering script) — https://cloud.r-project.org/
- **RStudio** (Recommended IDE for running OUKS scripts, which are adapted to the RStudio headings system) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
Rscript "Scripts (R)/6. Filtering.R"
```

## Evaluation signals

- Filtered peak table exists and parses as .csv in the OUKS Datasets (csv) folder format
- Feature count decreases relative to the imputed input table, with removal traceable to the quality-checking and D-Ratio criteria
- No remaining missing values in the retained feature matrix (input came from the imputation step)
- QC and batch metadata columns are consistent between the filtered table and metadata.csv so technical-variation accounting can be verified
- Downstream scripts (7. Normalization, 8. Grouping, 9. Statistics) run without errors on the filtered output

## Limitations

- The OUKS filtering step only defines the operation generically ('peaks filtering for quality checking and accounting of technical variation'); exact numeric D-Ratio thresholds are not stated in the provided context
- Some packages with strong version dependency are archived in the 'Required packages' folder; mismatched versions may hinder reproducibility
- Known bugs were fixed across the OUKS scripts (e.g. p.adjust in Normalization/Correction/Statistics); older versions may produce different filtering or downstream behavior
- The toolbox assumes familiarity with basic R syntax and a PC with Internet connection; Windows OS is desirable

## Evidence

- [methods] Filtering workflow step: "`6. Filtering.R` | peaks filtering for quality checking and accounting of technical variation"
- [discussion] D-Ratio filtering added: ""6. Filtering": D-Ratio filtering was added."
- [intro] Nine-step toolbox: "providing comprehensive nine step LC-MS untargeted metabolomic profiling data processing toolbox"
- [readme] Version-dependency limitation: "[Required packages](https://github.com/plyush1993/OUKS/tree/main/Required%20packages%20(archive)) section is for storing packages archives with strong version dependency."
- [readme] Downstream ordering: "`7. Normalization.R` | data normalization and adjusting of biological variation"
