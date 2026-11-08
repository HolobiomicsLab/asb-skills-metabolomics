---
name: exposome-compound-detection-benchmarking
description: Use when comparing compound-detection depth of JPA vs conventional peak
  picking on serially diluted LC-MS exposome standard mixture data (e.g. MTBLS2631
  RP(+) mzXML files), to reproduce or re-quantify detection fold-gains per dilution
  level.
when_to_use_negative:
- Processing full-scan or DIA data sets with the MR (MS2 recognition) step — the README
  explicitly warns not to use find.level3features on full-scan/DIA data
- Input is already a per-file aligned feature table from other software (use custom.featureTable
  instead of re-running XCMS.featureTable on raw data)
- Data are not serially diluted standards matched against a known compound list —
  per-compound recovery counts and fold gains are not computable
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package providing XCMS.featureTable, find.level3features (MR features),
    find.targetfeatures, and peak.alignment used to run both baseline and joint pipelines
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Runtime environment; JPA requires R version 4.0.0 or above
- name: devtools
  role: Used to install JPA from GitHub via install_github("HuanLab/JPA")
- name: BiocManager
  role: Installs Bioconductor dependencies (e.g. xcms) required by JPA
- name: xcms
  role: Embedded peak-picking engine used by XCMS.featureTable for the PP-only baseline
    and PP features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:8ca22ea048999ed6ba3c1cd7c56a15b1952c9b1cb6f97c9ce95a8651c16c3c2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - BiocManager
  - JPA
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# exposome-compound-detection-benchmarking

## Summary

Benchmarks JPA's joint feature extraction (peak picking + MS2 recognition + targeted extraction) against XCMS peak-picking-only by counting how many of the 505 exposome compounds (250 drugs + 255 pesticides) are recovered per concentration level in RP(+) mode, quantifying the reported ~2.3-fold detection gain.

## When to use

Use when comparing compound-detection depth of JPA vs conventional peak picking on serially diluted LC-MS exposome standard mixture data (e.g. MTBLS2631 RP(+) mzXML files), to reproduce or re-quantify detection fold-gains per dilution level.

## When NOT to use

- Processing full-scan or DIA data sets with the MR (MS2 recognition) step — the README explicitly warns not to use find.level3features on full-scan/DIA data
- Input is already a per-file aligned feature table from other software (use custom.featureTable instead of re-running XCMS.featureTable on raw data)
- Data are not serially diluted standards matched against a known compound list — per-compound recovery counts and fold gains are not computable

## Inputs

- RP(+) mzXML files of the five-level serially diluted exposome standard mixture (MTBLS2631 deposit)
- Targeted compound list (CSV) covering the 505 exposome compounds (250 drugs + 255 pesticides)
- R >= 4.0.0 environment with JPA installed from GitHub (HuanLab/JPA)

## Outputs

- Per-level counts of exposome compounds detected under PP-only vs full JPA
- Per-level and average fold-gain values (expected average ~2.3-fold)
- exposome_detection_counts.csv with per-level compound counts and fold gains

## How to apply

Obtain the RP(+) mzXML files of the five-level serially diluted exposome standard mixture (250 drugs + 255 pesticides) from the MTBLS2631 deposit and place them in a single folder. Run the PP-only baseline with XCMS.featureTable() using XCMS peak picking (ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100). Run the full JPA pipeline on the same data: find.level3features() with level3.threshold=2 to add MS2-recognized (MR) features, find.targetfeatures() with target.threshold=2 against the targeted compound list, and peak.alignment() with bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100. For each of the five concentration levels, match detected features against the 505-compound list and count compounds recovered under PP-only versus full JPA, then compute per-level fold gains and write them to exposome_detection_counts.csv.

## Related tools

- **JPA** (R package providing XCMS.featureTable, find.level3features (MR features), find.targetfeatures, and peak.alignment used to run both baseline and joint pipelines) — https://github.com/HuanLab/JPA.git
- **R** (Runtime environment; JPA requires R version 4.0.0 or above)
- **devtools** (Used to install JPA from GitHub via install_github("HuanLab/JPA"))
- **BiocManager** (Installs Bioconductor dependencies (e.g. xcms) required by JPA)
- **xcms** (Embedded peak-picking engine used by XCMS.featureTable for the PP-only baseline and PP features)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100); featureTable <- find.level3features(data=MSdata, level3.threshold=2)
```

## Evaluation signals

- Computed per-level compound counts and fold gains must reproduce the reported detection gains, including the average of 2.3-fold more exposure compounds detected by JPA vs PP-only
- The PP-only baseline must be generated with the exact XCMS parameters (ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100) so the comparison is like-for-like
- Features counted under full JPA must include PP, MR (level3.threshold=2), and targeted (target.threshold=2) levels, with duplicates not double-counted (targeted features already present as PP or MR are not re-added)
- Peak alignment on the multi-sample run must use bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100
- exposome_detection_counts.csv must contain one row per concentration level with PP-only count, full-JPA count, and fold gain, including the highest and lowest levels

## Limitations

- MR feature extraction must not be applied to full-scan or DIA data sets, per the README warning
- The MR threshold (level3.threshold=2) should be validated; the thresholdEstimate.R script on GitHub estimates true-positive thresholds
- JPA accepts raw data only in formats compatible with MS-Convert (mzXML); all mzXML files for a run must sit in one folder with no irrelevant files
- The 2.3-fold gain is specific to the exposome standard mixture benchmark; gains vary with data type (e.g. 25% of features rescued in serially diluted urine samples)

## Evidence

- [abstract] JPA detected an average of 2.3-fold more exposure compounds than conventional peak picking only: "JPA detected an average of 2.3-fold more exposure compounds than conventional peak picking only."
- [methods] find.level3features with level3.threshold: "featureTable <- find.level3features(data = MSdata, level3.threshold = 2)"
- [methods] peak.alignment parameters: "featureTable <- peak.alignment(data = MSdata, bw = 5, minfrac = 0.5, mzwid = 0.015,"
- [methods] targeted features not duplicated: "extract features with a targeted list. If these features already exists in the original feature table as either PP or MR features, they will not be added."
- [readme] XCMS.featureTable invocation and parameters: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,"
- [readme] MR step restriction to DDA data: "Please do not use this function when processing full-scan or DIA data set!"
- [full_text] targeted-list extraction purpose: "JPA performs feature extraction by combining a conventional peak picking algorithm and strategies for (1) recognizing features with bad peak shapes but that have tandem mass spectra (MS2)"
