---
name: fdr-multiple-testing-correction
description: Use when many features are tested independently (one test per metabolomic
  feature) and raw p-values must be FDR-adjusted — e.g. reproducing Fig. 3f-style
  significant-feature counts at FDR <= 0.2 on batch-corrected ST000868 data.
when_to_use_negative:
- Input is still raw LC/MS data (NetCDF/mzML files) — batch-aware preprocessing (cdf.to.ftr
  pipeline) must run first before any feature-level testing
- Post-hoc batch correction would be used to 'fix' misaligned feature tables — article
  notes such methods only adjust signal intensity and cannot correct peak alignment
  errors
- Only a single hypothesis test is being run, where no multiple-testing adjustment
  is needed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: apLCMS
  role: Produces the two-stage batch-aware feature tables whose per-feature t-test
    p-values are BH-adjusted and compared against traditional apLCMS/XCMS counts at
    FDR <= 0.2
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Environment in which per-feature two-sample t-tests and Benjamini-Hochberg
    p.adjust FDR correction are performed
- name: WaveICA
  role: Batch effect correction applied to ST000868 feature intensity matrices prior
    to feature-wise t-testing and FDR counting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:b6098df6aa62d0e27410aa9df212f82b87fef9e79dbb97c8e8167aa5d62d57cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - WaveICA
  - apLCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fdr-multiple-testing-correction

## Summary

Benjamini-Hochberg FDR correction over per-feature two-sample t-test p-values in LC/MS metabolomics feature tables, counting significant features at FDR <= 0.2 to compare preprocessing/batch-correction strategies (two-stage apLCMS vs. traditional apLCMS/XCMS with ComBat or WaveICA).

## When to use

Use when many features are tested independently (one test per metabolomic feature) and raw p-values must be FDR-adjusted — e.g. reproducing Fig. 3f-style significant-feature counts at FDR <= 0.2 on batch-corrected ST000868 data.

## When NOT to use

- Input is still raw LC/MS data (NetCDF/mzML files) — batch-aware preprocessing (cdf.to.ftr pipeline) must run first before any feature-level testing
- Post-hoc batch correction would be used to 'fix' misaligned feature tables — article notes such methods only adjust signal intensity and cannot correct peak alignment errors
- Only a single hypothesis test is being run, where no multiple-testing adjustment is needed

## Inputs

- Batch-corrected feature intensity matrix (samples x features, e.g. WaveICA- or ComBat-corrected ST000868 matrix)
- Sample annotation / cell-type group labels for the two-sample t-tests
- Vector of raw per-feature t-test p-values for BH adjustment

## Outputs

- Vector of BH-adjusted FDR q-values (one per feature)
- Count of features with FDR <= 0.2 per preprocessing/batch-correction setting
- Summary table, e.g. fig3f_significant_feature_counts.csv, and bar-chart figure mirroring Fig. 3f

## How to apply

Load the preprocessed, batch-corrected feature intensity matrix (samples x features with cell-type annotations, e.g. ST000868 corrected by WaveICA or ComBat). Run a two-sample t-test on every feature between the predefined sample groups (cell-type groups). Adjust all p-values jointly with the Benjamini-Hochberg procedure (R's p.adjust with method='BH'). Count features with adjusted FDR <= 0.2 per correction setting, compile counts into a summary table (fig3f_significant_feature_counts.csv), and compare against counts from traditionally preprocessed apLCMS and XCMS feature tables — the two-stage approach is expected to yield more significant features because it produces more consistent feature detection and quantification before testing.

## Related tools

- **apLCMS** (Produces the two-stage batch-aware feature tables whose per-feature t-test p-values are BH-adjusted and compared against traditional apLCMS/XCMS counts at FDR <= 0.2) — https://github.com/tianwei-yu/apLCMS
- **R** (Environment in which per-feature two-sample t-tests and Benjamini-Hochberg p.adjust FDR correction are performed)
- **WaveICA** (Batch effect correction applied to ST000868 feature intensity matrices prior to feature-wise t-testing and FDR counting)

## Examples

```
p.adj <- p.adjust(ttest.pvals, method="BH"); n.sig <- sum(p.adj <= 0.2)
```

## Evaluation signals

- Adjusted p-values are bounded in [0,1] and BH-adjusted p-values are >= the raw p-values for every feature
- Count of features with FDR <= 0.2 is finite, reproducible per parameter setting, and <= the number of tested features
- Two-stage apLCMS + batch correction yields a higher count of FDR <= 0.2 features than traditional apLCMS and XCMS (as in Fig. 3f)
- WaveICA-corrected results are similar to ComBat-corrected results in significant-feature counts
- Summary table exports the expected per-setting counts to fig3f_significant_feature_counts.csv

## Limitations

- Post-hoc batch effect correction (ComBat, WaveICA) can only adjust signal intensity; it cannot address misalignment of features across batches or incomplete weak signal recovery, so testing may be performed on suboptimal feature tables
- Treating all samples as a single batch during preprocessing can cause peak alignment errors that cannot be corrected post hoc before FDR testing
- Counts are dataset- and threshold-specific (ST000868, FDR <= 0.2) and may not transfer to other metabolomics datasets

## Evidence

- [other] Two-stage t-test/FDR workflow on ST000868: [evidence span withheld — anchor status: not_found]
- [other] BH threshold definition: [evidence span withheld — anchor status: not_found]
- [other] Fig. 3f comparative finding: [evidence span withheld — anchor status: not_found]
- [abstract] Better downstream analysis from two-stage preprocessing: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [intro] Scope of downstream correction limits: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches"
