---
name: lc-ms-feature-extraction
description: Use when processing raw mzXML LC-MS metabolomics or exposomics data (DDA
  mode) where low-abundant or bad-shape features are missed by conventional peak picking
  and deeper feature coverage is needed.
when_to_use_negative:
- Processing full-scan or DIA data — find.level3features relies on tandem MS2 spectra
  and the README explicitly warns not to use it for full-scan or DIA datasets
- Input is already an aligned feature table — custom.featureTable requires per-sample,
  unaligned CSV feature tables (one per mzXML file), not aligned tables
- Input data are not DDA, since MR feature rescue is only valid when MS2 spectra are
  acquired per cycle
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: JPA
  role: R package providing XCMS.featureTable, custom.featureTable, find.level3features,
    find.targetfeatures, and peak.alignment for joint PP + MR + targeted feature extraction
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded peak-picking engine used by JPA's XCMS.featureTable to extract MS1
    PP features
- name: devtools
  role: Used to install the JPA package from GitHub
- name: BiocManager
  role: CRAN bootstrap dependency installer for the JPA installation path
- name: MS-DIAL
  role: Alternative tool whose CSV feature tables can be converted with convertCSV.R
    and ingested via custom.featureTable
- name: MZmine 2
  role: Alternative tool whose pre-processed feature tables can be imported through
    custom.featureTable for comparison
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:de62c7d003f84204241bc7a4328e928e26a48e54553cab2b64b91fe077fd33bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - BiocManager
  - JPA
  - MS-DIAL
  - MZmine 2
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# lc-ms-feature-extraction

## Summary

Extract metabolic features from raw LC-MS data by combining conventional MS1 peak picking (PP) with JPA's MS2-recognition (MR) and targeted-list feature rescue, increasing chemical coverage beyond peak-picking-only workflows.

## When to use

Use when processing raw mzXML LC-MS metabolomics or exposomics data (DDA mode) where low-abundant or bad-shape features are missed by conventional peak picking and deeper feature coverage is needed.

## When NOT to use

- Processing full-scan or DIA data — find.level3features relies on tandem MS2 spectra and the README explicitly warns not to use it for full-scan or DIA datasets
- Input is already an aligned feature table — custom.featureTable requires per-sample, unaligned CSV feature tables (one per mzXML file), not aligned tables
- Input data are not DDA, since MR feature rescue is only valid when MS2 spectra are acquired per cycle

## Inputs

- mzXML file(s) from LC-MS analyses (any vendor format compatible with MS-Convert), placed in a folder with no other irrelevant mzXML files
- Optionally, per-sample CSV feature tables in mz, rt, rtmin, rtmax, intensity column order with retention times in seconds (e.g. converted from MS-DIAL via convertCSV.R)
- In-house targeted standard metabolite library CSV (e.g. LibraryCSVHILIC-.csv) for targeted feature extraction

## Outputs

- Dataframe-formatted feature table plus MSnbase object with columns mz, rt, rtmin, rtmax, maxo, sample, level (PP/MR/target)
- Aligned multi-sample feature table after peak.alignment
- Per-level feature counts (PP vs MR vs target) for coverage assessment

## How to apply

Install the JPA R package from GitHub with devtools (R >= 4.0.0), then extract MS1 PP features from mzXML files using XCMS.featureTable with parameters such as mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100. Next run find.level3features(level3.threshold=2) to rescue MR features recognized by their tandem MS2 spectra despite poor peak shapes — this step is only valid for DDA data, not full-scan or DIA. Optionally add targeted-list features with find.targetfeatures(target.threshold=2) against an in-house standard library CSV (e.g. LibraryCSVHILIC-.csv); features already present as PP or MR will not be re-added. For multi-sample studies, align features afterwards with peak.alignment(bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100, quantitative.method='maxo'). The 'level' column in the output featureTable distinguishes PP, MR, and target features so rescue counts can be quantified.

## Related tools

- **JPA** (R package providing XCMS.featureTable, custom.featureTable, find.level3features, find.targetfeatures, and peak.alignment for joint PP + MR + targeted feature extraction) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded peak-picking engine used by JPA's XCMS.featureTable to extract MS1 PP features)
- **devtools** (Used to install the JPA package from GitHub)
- **BiocManager** (CRAN bootstrap dependency installer for the JPA installation path)
- **MS-DIAL** (Alternative tool whose CSV feature tables can be converted with convertCSV.R and ingested via custom.featureTable)
- **MZmine 2** (Alternative tool whose pre-processed feature tables can be imported through custom.featureTable for comparison)

## Examples

```
featureTable <- XCMS.featureTable(dir=dir, mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data=MSdata, level3.threshold=2)
```

## Evaluation signals

- The 'level' column of featureTable tags every feature as PP, MR, or target; verify all three levels are populated and no targeted feature duplicates an existing PP or MR feature
- JPA should extract more features than XCMS peak picking alone; the paper reports JPA rescued an average of 25% of features missed by peak picking and detected 2.3-fold more exposure compounds
- Across serial dilutions, PP feature counts decrease with dilution while MR-rescued features compensate — check the MR count does not collapse to zero on dilution series
- For DDA cycle-time tests (1, 2, 3 s from MTBLS2631), JPA-PP counts should decrease and JPA-MR counts should increase monotonically with cycle time as in Figure S3
- MR feature threshold can be justified by running the thresholdEstimate.R script from the GitHub repository before choosing level3.threshold

## Limitations

- MR feature extraction (find.level3features) must not be used on full-scan or DIA data since it depends on MS2 spectra
- The level3.threshold (e.g. 2) and target.threshold must be estimated; JPA provides thresholdEstimate.R for this purpose
- Low-abundant or poor-shape features missed by conventional peak picking are only recoverable in DDA mode where MS2 spectra were collected
- Custom feature tables must be unaligned, one CSV per mzXML file, in exact mz, rt, rtmin, rtmax, intensity column order with retention times in seconds

## Evidence

- [abstract] JPA performs feature extraction by combining a conventional peak picking algorithm and strategies for (1) recognizing features with bad peak shapes but that have tandem mass spectra (MS2): "JPA performs feature extraction by combining a conventional peak picking algorithm and strategies for (1) recognizing features with bad peak shapes but that have tandem mass spectra (MS2)"
- [methods] The XCMS.featureTable() function outputs a dataframe formatted featuretable: "The XCMS.featureTable() function outputs a dataframe formatted featuretable"
- [methods] extract features with a targeted list. If these features already exists in the original feature table as either PP or MR features, they will not be added.: "extract features with a targeted list. If these features already exists in the original feature table as either PP or MR features, they will not be added."
- [readme] All mzXML file(s) need to be placed in the same folder containing no other irrelevant mzXML files.: "All mzXML file(s) need
    to be placed in the same folder containing no other irrelevant mzXML
    files."
- [readme] Please do not use this function when processing full-scan or DIA data set!: "This step is optional. Please do not use this function when processing full-scan or DIA data set!"
- [abstract] JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
