---
name: feature-level-count-tabulation
description: Use when a JPA-processed feature table contains a 'level' column labeling
  features as PP or MR, and you need per-level counts to compare feature depth across
  datasets (e.g., 1 s vs 2 s vs 3 s DDA cycle times) or to quantify MR-rescued features.
when_to_use_negative:
- Feature tables were produced by custom.featureTable() or other software without
  a subsequent find.level3features() call, so no MR-level features exist and PP-vs-MR
  comparison is meaningless
- Input data are full-scan or DIA rather than DDA — JPA explicitly warns not to run
  the MS2-recognition step, so level counts would not reflect MR rescue
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: JPA
  role: R package whose XCMS.featureTable() and find.level3features() functions produce
    the level-annotated feature table ('PP' and 'MR' features) that is counted; installed
    via install_github('HuanLab/JPA')
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded peak-picking engine inside JPA that generates the PP features counted
    in the summary table
- name: R
  role: Runtime environment (R >= 4.0.0, RStudio recommended) in which the feature
    table is loaded and level counts are tabulated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:853186d54264e897a507643366edbfe62000910c2e3bbbbd7a7b6f9ec7a0a125
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# feature-level-count-tabulation

## Summary

Tabulates metabolic feature counts by the JPA 'level' column (PP vs MR) after joint feature extraction, enabling robustness comparisons across acquisition conditions such as DDA cycle times. It quantifies how much the MS2-recognition (MR) strategy rescues features missed by conventional XCMS peak picking.

## When to use

Use when a JPA-processed feature table contains a 'level' column labeling features as PP or MR, and you need per-level counts to compare feature depth across datasets (e.g., 1 s vs 2 s vs 3 s DDA cycle times) or to quantify MR-rescued features.

## When NOT to use

- Feature tables were produced by custom.featureTable() or other software without a subsequent find.level3features() call, so no MR-level features exist and PP-vs-MR comparison is meaningless
- Input data are full-scan or DIA rather than DDA — JPA explicitly warns not to run the MS2-recognition step, so level counts would not reflect MR rescue

## Inputs

- JPA feature table (dataframe with columns mz, rt, rtmin, rtmax, maxo, sample, level) output by XCMS.featureTable() plus find.level3features()
- mzXML folders of DDA acquisitions at each cycle time (e.g., from MetaboLights MTBLS2631), each dataset in its own folder

## Outputs

- Summary table of feature counts per 'level' value (PP, MR) per dataset/cycle time
- Trend validation flag for any cycle time whose counts violate the expected monotonic PP-decrease/MR-increase pattern

## How to apply

After running XCMS.featureTable() on each mzXML folder (e.g., ppm = 10, peakwidth = c(5, 20), snthresh = 6, prefilter = c(3, 100)) and find.level3features(data = MSdata, level3.threshold = 2) to append MR features, group the resulting dataframe by the 'level' column and count rows per level. Assemble a summary table of PP and MR counts per dataset/cycle time. The 'level' column is emitted by JPA's feature-table functions, so no extra parsing is needed. For robustness studies, check monotonic trends: JPA-PP counts should decrease and JPA-MR counts should increase as DDA cycle time lengthens from 1 s to 3 s, flagging any cycle time that violates this pattern.

## Related tools

- **JPA** (R package whose XCMS.featureTable() and find.level3features() functions produce the level-annotated feature table ('PP' and 'MR' features) that is counted; installed via install_github('HuanLab/JPA')) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded peak-picking engine inside JPA that generates the PP features counted in the summary table)
- **R** (Runtime environment (R >= 4.0.0, RStudio recommended) in which the feature table is loaded and level counts are tabulated)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, ppm=10, peakwidth=c(5,20), snthresh=6, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data = MSdata, level3.threshold = 2); table(featureTable$level)
```

## Evaluation signals

- The feature table contains a 'level' column whose values include 'PP' (from peak picking) and 'MR' (from MS2 recognition); count PP + MR equals total table rows
- For a cycle-time series, JPA-PP counts decrease monotonically from 1 s to 3 s while JPA-MR counts increase monotonically, matching Figure S3; flag any violating cycle time
- MR features only appear after find.level3features() has been run — an unrescued XCMS.featureTable() output should contain only 'PP' rows
- MR threshold used is level3.threshold = 2, valid only for DDA data; verify data mode before trusting MR counts

## Limitations

- MR rescue is valid only for DDA data; JPA's manual warns 'Please do not use this function when processing full-scan or DIA data set!'
- For multi-sample analyses, sample alignment is performed after feature extraction, so per-level counts from individual feature tables may include redundant features (e.g., near-duplicate mz entries within a single sample) before alignment
- Conventional peak picking (and thus PP counts) misses features with low signal intensities, poor chromatographic peak shapes, or those not fitting parameter settings — PP counts alone understate chemical coverage

## Evidence

- [readme] The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature.: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [methods] Find MR features and add them to the original feature table.: "Find MR features and add them to the original feature table."
- [readme] Please do not use this function when processing full-scan or DIA data set!: "This step is optional. Please do not use this function when processing full-scan or DIA data set!"
- [abstract] JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
