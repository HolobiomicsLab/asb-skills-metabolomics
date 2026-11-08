---
name: targeted-feature-extraction-from-standard-library
description: Use when serial-dilution or low-abundance analyte features are missing
  from a PP (peak-picking-only) feature table but their expected m/z and retention
  times are known from an in-house standard library CSV, and the goal is to quantify
  them (e.g., for LOD determination).
when_to_use_negative:
- No in-house targeted list (standard library CSV) exists for the metabolites of interest
  — find.targetfeatures is list-driven and cannot rescue unknown features (use find.level3features/MR
  instead)
- The input is an already-aligned feature table or MS-DIAL output CSV without conversion
  — JPA requires per-sample CSV feature tables, since 'Aligned feature table is not
  acceptable'
- The features of interest are already present in the table as PP or MR features —
  they will not be re-added, so running targeted extraction adds nothing
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: JPA
  role: R package providing find.targetfeatures() for targeted-list feature extraction
    from raw mzXML data against a standard library
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded XCMS peak-picking functions supply the initial PP feature table that
    targeted extraction complements
- name: devtools
  role: Used to install the JPA package from GitHub (install_github("HuanLab/JPA"))
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:e19aebaf3b269d84749096fc44d6180f45645fd42fca07b9db406f00db09cd6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# targeted-feature-extraction-from-standard-library

## Summary

JPA's targeted-list extraction (find.targetfeatures) rescues metabolite features that conventional XCMS peak picking missed by re-interrogating raw mzXML data against an in-house standard metabolite library (CSV of m/z and retention time), adding them to the feature table labeled as a distinct level.

## When to use

Use when serial-dilution or low-abundance analyte features are missing from a PP (peak-picking-only) feature table but their expected m/z and retention times are known from an in-house standard library CSV, and the goal is to quantify them (e.g., for LOD determination).

## When NOT to use

- No in-house targeted list (standard library CSV) exists for the metabolites of interest — find.targetfeatures is list-driven and cannot rescue unknown features (use find.level3features/MR instead)
- The input is an already-aligned feature table or MS-DIAL output CSV without conversion — JPA requires per-sample CSV feature tables, since 'Aligned feature table is not acceptable'
- The features of interest are already present in the table as PP or MR features — they will not be re-added, so running targeted extraction adds nothing

## Inputs

- Raw LC-MS mzXML file(s) in a dedicated folder (any vendor format convertible via MS-Convert), per mode (HILIC(-), RP(+))
- In-house standard metabolite library in CSV format (e.g., LibraryCSVHILIC-.csv) with m/z and retention time of targeted metabolites
- Existing JPA featureTable dataframe and MSnbase object from XCMS.featureTable / find.level3features

## Outputs

- Updated featureTable dataframe with targeted-level features added (level column distinguishes PP, MR, and targeted features)
- Aligned feature table across the dilution series for downstream calibration-curve and LOD computation (e.g., lod_comparison.csv ratios)

## How to apply

After extracting PP features with XCMS.featureTable (e.g., mz.tol=10, ppm=10, peakwidth=c(5,20), snthresh=6, prefilter=c(3,100), noise=100) and optionally MR features with find.level3features(level3.threshold=2), run find.targetfeatures on the MSnbase data object, pointing tarFTdir/tarFTname at the in-house standard library CSV (e.g., LibraryCSVHILIC-.csv for HILIC(-) mode) with target.threshold=2. The function extracts features matching the targeted list directly from the raw mzXML files; if a target already exists in the table as a PP or MR feature it is not added again, so the table gains only genuinely rescued low-intensity or bad-shape peaks. Run this per polarity/mode (HILIC(-) vs RP(+)) with the matching library file, then align all feature levels across the dilution series with peak.alignment (bw=5, minfrac=0.5, mzwid=0.015, quantitative.method="maxo") before building calibration curves. The rationale is that JPA's targeted strategy recovers features conventional algorithms miss due to low signal intensity or poor chromatographic peak shape, yielding LODs up to thousands of folds lower.

## Related tools

- **JPA** (R package providing find.targetfeatures() for targeted-list feature extraction from raw mzXML data against a standard library) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded XCMS peak-picking functions supply the initial PP feature table that targeted extraction complements)
- **devtools** (Used to install the JPA package from GitHub (install_github("HuanLab/JPA")))

## Examples

```
featureTable <- find.targetfeatures(data = MSdata, tarFTdir = tarFTdir, tarFTname = "LibraryCSVHILIC-.csv", target.threshold = 2)
```

## Evaluation signals

- The output featureTable's 'level' column contains a distinct level for targeted features, and PP/MR features are not duplicated by targeted extraction
- Targeted features match library entries by both m/z and retention time from the targeted list when matched to the calibration series
- In the serial-dilution standard-mixture validation, per-metabolite LOD ratios (PP-only LOD / full-JPA LOD) reach the reported magnitudes — up to 3425-fold in HILIC(-) and 15,074-fold in RP(+) mode
- Rescued features show acceptable chromatographic peak shapes, analytical accuracy, and precision as evaluated in the paper
- target.threshold=2 is applied (with MR threshold estimable via the thresholdEstimate.R script on GitHub)

## Limitations

- If targeted features already exist in the feature table as PP or MR features they will not be added, so targeted extraction only rescues genuinely missed features
- Targeted extraction depends on the coverage and correctness of the in-house standard library CSV per ionization mode; a wrong-mode library (e.g., positive-mode library for HILIC(-) data) will fail to match
- MR-based feature extraction should not be used on full-scan or DIA datasets (README caution for the adjacent MS2-recognition step); targeted extraction must be run per polarity/mode separately

## Evidence

- [methods] targeted list extraction is list-driven against the in-house standard library: [evidence span withheld — anchor status: not_found]
- [methods] find.targetfeatures call with threshold 2: "featureTable <- find.targetfeatures(data = MSdata, tarFTdir = tarFTdir, tarFTname = tarFTname, target.threshold = 2)"
- [methods] no duplicate addition of existing features: "extract features with a targeted list. If these features already exists in the original feature table as either PP or MR features, they will not be added."
- [abstract] rationale: conventional peak picking misses low-intensity/poor-shape features: "Conventional feature extraction algorithms fail to recognize features with low signal intensities, poor chromatographic peak shapes, or those that do not fit the parameter settings."
- [readme] JPA requires unaligned per-sample input tables: "For multi-sample analysis, a CSV feature table for each single mzXML file needs to be provided. Aligned feature table is not acceptable."
