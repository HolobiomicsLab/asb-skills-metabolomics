---
name: peak-picking-parameter-tuning
description: Use when raw LC-MS mzXML files are being processed with XCMS.featureTable
  and you need to test how changing mass tolerance or S/N thresholds changes feature
  counts and peak-shape fidelity.
when_to_use_negative:
- When evaluating LOD gains or targeted rescue of the full JPA pipeline rather than
  the effect of peak-picking thresholds on feature counts and fidelity
- When raw vendor data must first be converted — mzXML files must already exist and
  the folder must contain no irrelevant mzXML files
- When a pre-existing feature table from other software is used instead of raw mzXML
  input
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: JPA
  role: Provides XCMS.featureTable() and find.level3features() used to re-extract
    PP and MR features under default and tolerant peak-picking parameters
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded XCMS peak-picking algorithms whose parameters (ppm, peakwidth, snthresh,
    mzdiff, prefilter, noise) are being tuned
  repo: https://rdrr.io/bioc/xcms/man/
- name: R
  role: Runtime language; JPA requires R version 4.0.0 or above
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:391f9073382fbc80134004b83ce61a2cef5c5220547bc52a58d39cb16d915a2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# peak-picking-parameter-tuning

## Summary

Tunes the JPA-embedded XCMS peak-picking parameters (ppm, snthresh, peakwidth) on raw LC-MS mzXML data and quantifies how looser vs default thresholds shift PP/MR feature counts and feature fidelity rates.

## When to use

Use when raw LC-MS mzXML files are being processed with XCMS.featureTable and you need to test how changing mass tolerance or S/N thresholds changes feature counts and peak-shape fidelity.

## When NOT to use

- When evaluating LOD gains or targeted rescue of the full JPA pipeline rather than the effect of peak-picking thresholds on feature counts and fidelity
- When raw vendor data must first be converted — mzXML files must already exist and the folder must contain no irrelevant mzXML files
- When a pre-existing feature table from other software is used instead of raw mzXML input

## Inputs

- Folder of mzXML files (MS-Convert-compatible vendor format, e.g. NISTplasmaDDARP(-) mzXML samples)
- R >= 4.0.0 environment with the JPA package installed via devtools

## Outputs

- Feature tables (dataframe with mz, rt, rtmin, rtmax, maxo, sample, level columns) for default and tolerant parameter settings
- Per-level (PP/MR) feature counts and fidelity rates for each parameter setting
- Side-by-side comparison table, e.g. fidelity_comparison_table.csv

## How to apply

Run XCMS.featureTable() on the mzXML folder with JPA's default parameters (mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100) to build the baseline PP feature table, then add MR features via find.level3features(level3.threshold=2). Reprocess the identical raw files with tolerant thresholds (mass tolerance 30 ppm, S/N 3) and repeat both the PP and MR steps. Count features per level ('PP' and 'MR') in each setting and compute the fidelity rate, i.e., the proportion of features with acceptable chromatographic peak shapes, using the Figure S4 evaluation protocol. The rationale is that loosening thresholds trades recall for precision: JPA-PP gains features but its fidelity rate drops (~10% on average), while JPA-MR features keep a similar fidelity rate, so many features still require JPA rescue. Tabulate counts and fidelity rates side-by-side (e.g., fidelity_comparison_table.csv) to decide which parameter setting to carry forward.

## Related tools

- **JPA** (Provides XCMS.featureTable() and find.level3features() used to re-extract PP and MR features under default and tolerant peak-picking parameters) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded XCMS peak-picking algorithms whose parameters (ppm, peakwidth, snthresh, mzdiff, prefilter, noise) are being tuned) — https://rdrr.io/bioc/xcms/man/
- **R** (Runtime language; JPA requires R version 4.0.0 or above)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm = 10, peakwidth = c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)
```

## Evaluation signals

- Default run should reproduce JPA defaults: ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100; tolerant run uses 30 ppm and S/N 3
- Under tolerant thresholds, PP feature counts increase while MR feature counts decrease slightly relative to the default run
- JPA-PP fidelity rate drops by roughly 10% on average under tolerant thresholds, while JPA-MR fidelity stays similar
- Feature table 'level' column must correctly label features as 'PP' or 'MR' in both runs
- Comparison table rows pair default vs tolerant settings with counts and fidelity rates per level

## Limitations

- Conventional peak picking fails on features with low signal intensities, poor chromatographic peak shapes, or parameters that do not fit the settings, so parameter tuning cannot recover all missed features
- find.level3features must not be used on full-scan or DIA data sets (per the JPA README)
- The MR-feature threshold (level3.threshold) should be estimated with the thresholdEstimate.R script provided on the JPA GitHub

## Evidence

- [abstract] threshold sensitivity rationale: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
- [readme] default XCMS.featureTable parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,
                             snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [readme] XCMS.featureTable output format: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object.
# The "level" column shows the level of each feature."
- [methods] PP feature extraction via embedded XCMS: [evidence span withheld — anchor status: not_found]
- [readme] MR feature threshold guidance: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
