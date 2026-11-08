---
name: targeted-feature-rescue-from-target-list
description: Use when processing raw LC-MS mzXML data against a known compound list
  (e.g. a 505-compound drug/pesticide exposome mixture) and conventional peak picking
  misses expected targets, especially at low concentration levels.
when_to_use_negative:
- The input is already a curated feature table from other software rather than raw
  mzXML data (use custom.featureTable instead, though targeted extraction still applies)
- The analysis is full-scan or DIA data with no targeted compound list available —
  find.targetfeatures() requires a target list and MR extraction should not be used
  on full-scan/DIA data
- The goal is de novo discovery of unknown features rather than confirming/recovering
  known exposomic compounds
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: Provides find.targetfeatures() to extract target-list features plus XCMS.featureTable()
    and peak.alignment() used in the surrounding pipeline
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Runtime environment for JPA (version 4.0.0 or above required)
- name: devtools
  role: Used to install JPA from GitHub via install_github("HuanLab/JPA")
- name: BiocManager
  role: Prerequisite package installer for the JPA/xcms dependency stack
- name: xcms
  role: Embedded peak-picking engine generating the PP baseline feature table that
    the targeted list supplements
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:88ce0f50dbb770e484f787ba14531750e1ec74adb8d58ace5830a0c3540efd06
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# targeted-feature-rescue-from-target-list

## Summary

Rescues exposomic/metabolomic features missed by conventional XCMS peak picking by matching raw LC-MS data against a targeted compound list with JPA's find.targetfeatures(), lifting detection depth for low-abundant or badly-shaped peaks.

## When to use

Use when processing raw LC-MS mzXML data against a known compound list (e.g. a 505-compound drug/pesticide exposome mixture) and conventional peak picking misses expected targets, especially at low concentration levels.

## When NOT to use

- The input is already a curated feature table from other software rather than raw mzXML data (use custom.featureTable instead, though targeted extraction still applies)
- The analysis is full-scan or DIA data with no targeted compound list available — find.targetfeatures() requires a target list and MR extraction should not be used on full-scan/DIA data
- The goal is de novo discovery of unknown features rather than confirming/recovering known exposomic compounds

## Inputs

- Raw LC-MS mzXML files (MS-Convert-compatible vendor data), all in one folder (e.g. MTBLS2631 RP(+) exposome standard mixture)
- Targeted compound list CSV (tarFTname, e.g. exposome list of 250 drugs + 255 pesticides) placed in tarFTdir
- PP feature table produced by JPA's XCMS.featureTable() (dataframe with mz, rt, rtmin, rtmax, maxo, sample, level columns)

## Outputs

- Expanded featureTable dataframe with target-list-rescued features added (labeled as target-level features, distinct from PP/MR)
- Aligned feature table after peak.alignment() across the five concentration levels
- Per-level compound recovery counts vs the 505-compound list (e.g. exposome_detection_counts.csv) showing fold-gain over PP-only

## How to apply

Place all mzXML files (e.g. the five-level serially diluted MTBLS2631 exposome standard, 250 drugs + 255 pesticides, RP(+) mode) in a single folder and run JPA's XCMS.featureTable() to obtain PP features (ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100). Then call find.targetfeatures() with target.threshold=2 against the targeted compound list in CSV format (e.g. LibraryCSV-like tarFTname), which extracts features matching known m/z-RT targets that were missed by PP; targets already present as PP or MR features are not duplicated. Optionally run find.level3features() with level3.threshold=2 first to add MS2-recognized (MR) features, then align samples across levels with peak.alignment(bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100). The rationale is that conventional algorithms fail on features with low signal intensities or poor/non-Gaussian peak shapes, and a targeted list recovers them by direct m/z and retention-time matching rather than shape-based peak detection.

## Related tools

- **JPA** (Provides find.targetfeatures() to extract target-list features plus XCMS.featureTable() and peak.alignment() used in the surrounding pipeline) — https://github.com/HuanLab/JPA.git
- **R** (Runtime environment for JPA (version 4.0.0 or above required))
- **devtools** (Used to install JPA from GitHub via install_github("HuanLab/JPA"))
- **BiocManager** (Prerequisite package installer for the JPA/xcms dependency stack)
- **xcms** (Embedded peak-picking engine generating the PP baseline feature table that the targeted list supplements)

## Examples

```
featureTable <- find.targetfeatures(data = MSdata, tarFTdir = tarFTdir, tarFTname = "LibraryCSV.csv", target.threshold = 2)
```

## Evaluation signals

- Match detected features at each of the five concentration levels against the 505-compound target list and confirm full-JPA counts exceed PP-only counts at every level, including the highest and lowest levels
- Compute fold-gain (JPA vs PP-only) across levels; the average should reproduce ~2.3-fold more exposure compounds detected
- Verify no duplicate features: compounds already present as PP or MR features must not be re-added by find.targetfeatures()
- Check the featureTable 'level' column to confirm rescued features are labeled with the targeted-list level and carry sane mz/rt/rtmin/rtmax values in seconds
- Validate alignment parameters took effect (bw=5, minfrac=0.5, mzwid=0.015, minsamp=1, max=100) so the same compound is consolidated across the five dilution levels

## Limitations

- Conventional feature extraction fails to recognize features with low signal intensities, poor chromatographic peak shapes, or those not fitting parameter settings — targeted rescue is bounded by what the target list contains
- MR-feature extraction must not be used on full-scan or DIA data sets (README warning); target-based extraction assumes a trusted compound list with accurate m/z and RT
- The target.threshold (e.g. 2) should be estimated with the thresholdEstimate.R helper; mis-set thresholds can admit false positives

## Evidence

- [abstract] Targeted-list rescue rationale: "JPA detected an average of 2.3-fold more exposure compounds than conventional peak picking only."
- [abstract] Why rescue is needed: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
- [methods] Targeted extraction deduplication: "extract features with a targeted list. If these features already exists in the original feature table as either PP or MR features, they will not be added."
- [methods] find.targetfeatures invocation: "featureTable <- find.targetfeatures(data = MSdata, tarFTdir = tarFTdir, tarFTname = tarFTname, target.threshold = 2)"
- [readme] README DIA/full-scan warning: "This step is optional. Please do not use this function when processing full-scan or DIA data set!"
- [readme] PP baseline parameters: "XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,
                             snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
