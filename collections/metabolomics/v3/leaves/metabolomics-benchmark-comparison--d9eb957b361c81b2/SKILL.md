---
name: metabolomics-benchmark-comparison
description: Use when comparing LC-MS metabolomics feature-extraction/annotation software
  head-to-head on the same raw or pre-processed data with pre-optimized parameters,
  e.g. urine RP(+) feature counts across JPA vs MZmine 2, XCMS, MS-DIAL.
when_to_use_negative:
- The input is a single pre-existing aligned feature table and no raw mzXML data or
  competing tools are involved — JPA requires per-file unaligned CSV tables and this
  skill is specifically a multi-tool benchmark.
- The data are full-scan or DIA acquisitions — the README warns not to use the MS2-recognition
  (MR) extraction step in that case, which is part of the benchmarked JPA workflow.
- The goal is quantitative validation (LOD, accuracy, precision) rather than sensitivity/coverage
  comparison of feature and annotation counts.
tools:
- name: JPA
  role: primary tool being benchmarked; performs PP/MR/target feature extraction,
    peak.alignment, adduct.isotope.annotation, ms2.tofeaturetable, and feature.annotation
    (dp=0.1)
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: embedded peak-picking engine behind XCMS.featureTable, used for JPA's PP features
    and as one of the three comparison tools
- name: MZmine 2
  role: comparison tool; its pre-processed feature table is imported into JPA via
    custom.featureTable for count comparison
- name: MS-DIAL
  role: comparison tool; its CSV output is converted with convertCSV.R before import
    via custom.featureTable
- name: devtools
  role: installs the JPA R package from GitHub (install_github('HuanLab/JPA'))
  repo: https://github.com/HuanLab/JPA.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:bfa90cf04e5043456980041ffc66dd6e1836d1ff5a2f62c717f8bbd9586b5f35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - MS-DIAL
  - MS-FINDER
  - MZmine
  - MZmine 2
  - MetaboAnalystR 4.0
  - ProteoWizard
  - R
  - SIRIUS
  - devtools
  - xcms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- benchmark-tool-comparison-metabolomics
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metabolomics-benchmark-comparison

## Summary

Benchmark the JPA joint feature-extraction and annotation workflow against MZmine 2, XCMS, and MS-DIAL on pre-optimized LC-MS data by tabulating extracted features (PP, MR, target levels) and annotated metabolites per tool.

## When to use

Use when comparing LC-MS metabolomics feature-extraction/annotation software head-to-head on the same raw or pre-processed data with pre-optimized parameters, e.g. urine RP(+) feature counts across JPA vs MZmine 2, XCMS, MS-DIAL.

## When NOT to use

- The input is a single pre-existing aligned feature table and no raw mzXML data or competing tools are involved — JPA requires per-file unaligned CSV tables and this skill is specifically a multi-tool benchmark.
- The data are full-scan or DIA acquisitions — the README warns not to use the MS2-recognition (MR) extraction step in that case, which is part of the benchmarked JPA workflow.
- The goal is quantitative validation (LOD, accuracy, precision) rather than sensitivity/coverage comparison of feature and annotation counts.

## Inputs

- urine RP(+) LC-MS mzXML raw data files
- pre-processed MZmine 2 feature table (CSV)
- MS-DIAL CSV output (converted via convertCSV.R)
- in-house targeted metabolite library CSV (e.g. LibraryCSVHILIC-.csv format)
- standard MS2 spectral library (.msp)

## Outputs

- per-tool feature counts broken down by JPA level (PP, MR, target)
- per-tool annotated metabolite counts
- JPA-aligned feature table with MS2 annotations and adduct/isotope annotations
- benchmark comparison table quantifying JPA's sensitivity advantage

## How to apply

Process the same urine RP(+) mzXML files with each tool using pre-optimized settings so the comparison isolates extraction/annotation capability rather than parameter tuning. For the JPA arm, extract MS1 PP features with XCMS.featureTable (mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100), add MR features via find.level3features (level3.threshold=2), add target features via find.targetfeatures (target.threshold=2) against the in-house library CSV, align with peak.alignment (bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100, quantitative.method='maxo'), assign MS2 spectra with ms2.tofeaturetable, and annotate against the standard .msp library with feature.annotation (dp=0.1). For MZmine 2 and MS-DIAL, import their pre-processed feature tables through custom.featureTable, first converting MS-DIAL CSV output with the provided convertCSV.R so columns follow the mz, rt, rtmin, rtmax, intensity order with retention times in seconds; XCMS output is generated directly. Tabulate feature counts by level (PP/MR/target) and annotated metabolite counts for all four tools and compare against the paper's expectation that JPA finds more features/metabolites (2.3-fold more exposure compounds than peak picking alone).

## Related tools

- **JPA** (primary tool being benchmarked; performs PP/MR/target feature extraction, peak.alignment, adduct.isotope.annotation, ms2.tofeaturetable, and feature.annotation (dp=0.1)) — https://github.com/HuanLab/JPA.git
- **xcms** (embedded peak-picking engine behind XCMS.featureTable, used for JPA's PP features and as one of the three comparison tools)
- **MZmine 2** (comparison tool; its pre-processed feature table is imported into JPA via custom.featureTable for count comparison)
- **MS-DIAL** (comparison tool; its CSV output is converted with convertCSV.R before import via custom.featureTable)
- **devtools** (installs the JPA R package from GitHub (install_github('HuanLab/JPA'))) — https://github.com/HuanLab/JPA.git

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)
```

## Evaluation signals

- JPA's total extracted feature count and annotated metabolite count exceed those of XCMS, MZmine 2, and MS-DIAL on the same urine RP(+) data, consistent with the paper's reported ~2.3-fold more exposure compounds versus peak picking alone.
- Imported MS-DIAL CSVs verify as having exactly the columns mz, rt, rtmin, rtmax, intensity with all retention times in seconds and one per-file CSV (unaligned) per mzXML file.
- The JPA feature table contains a 'level' column distinguishing PP, MR, and target features, with target features added only if absent as PP or MR.
- Alignment completes with the specified parameters (bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100, quantitative.method='maxo') and the aligned table carries maxo-style intensities.
- Annotation counts are reproducible at the fixed matching threshold dp=0.1 against the standard .msp library.

## Limitations

- MR feature extraction must not be run on full-scan or DIA data sets (README warning).
- Custom feature tables must be unaligned, one CSV per mzXML file, with RT columns in seconds; aligned tables are rejected.
- Conventional peak picking alone misses features with low signal intensities, poor peak shapes, or settings mismatches — which is precisely the bias being measured, so parameter pre-optimization for every tool is required for a fair comparison.
- MR threshold values (level3.threshold=2, target.threshold=2) should be justified with the provided thresholdEstimate.R code.

## Evidence

- [abstract] JPA detected an average of 2.3-fold more exposure compounds than conventional peak picking only.: "JPA detected an average of 2.3-fold more exposure compounds than conventional peak picking only."
- [abstract] Developed to comprehensively extract metabolic features from raw LC-MS data: "we developed an R package, JPA (short for Joint Metabolomic Data Processing and Annotation), to comprehensively extract metabolic features from raw LC-MS data"
- [methods] The custom.featureTable() function accepts external software feature tables: "The custom.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object."
- [readme] README on MS2 recognition threshold and scope: "The threshold used for determining whether the MR features are true positive can be estimated by using the R code "thresholdEstimate.R" provided on the GitHub."
- [readme] README on input feature table format requirements: "The input feature table contain only columns in the following order: m/z, retention time, min retention time, max retention time, intensity."
