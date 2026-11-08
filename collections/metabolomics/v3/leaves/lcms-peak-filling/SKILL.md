---
name: lcms-peak-filling
description: Use when an XCMS xcmsSet has missing peaks after grouping and you need
  filled feature intensities, especially for features flagged misaligned by ncGTW,
  to compare CV before vs after realignment.
when_to_use_negative:
- The feature is already well aligned and was not flagged by the ncGTW misalignment
  detection module — peak-filling gains only apply to misaligned true positives
- Only the misalignment detection or realignment step is needed (tasks covered by
  the ncGTW detection/alignment skills), not intensity retrieval
- No raw LC-MS profiles are available — XCMS peak-filling requires the original files
  to extract the missing peaks
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: XCMS
  role: Provides fillPeaks and the fillPeaksChromPar routine that performs peak-filling
    after grouping using either its original warping functions or substituted ncGTW
    warping functions
- name: ncGTW
  role: Supplies per-sample realignment warping functions for substitution into fillPeaksChromPar
    and the compCV function used to compute per-feature coefficients of variation
    for quality comparison
  repo: https://github.com/ChiungTingWu/ncGTW
- name: R
  role: Execution environment (Bioconductor package ecosystem) in which the fillPeaks
    baseline/realigned runs and the compCV comparison are scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:68a17930394f620913be7a348c8f4384a555d340d7a3c1c4ae22e6220a9a269e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lcms-peak-filling

## Summary

Runs XCMS peak-filling to retrieve missing peaks after RT alignment, optionally substituting ncGTW per-sample realignment warping functions for XCMS's single warping function to improve filled-intensity quality for misaligned features.

## When to use

Use when an XCMS xcmsSet has missing peaks after grouping and you need filled feature intensities, especially for features flagged misaligned by ncGTW, to compare CV before vs after realignment.

## When NOT to use

- The feature is already well aligned and was not flagged by the ncGTW misalignment detection module — peak-filling gains only apply to misaligned true positives
- Only the misalignment detection or realignment step is needed (tasks covered by the ncGTW detection/alignment skills), not intensity retrieval
- No raw LC-MS profiles are available — XCMS peak-filling requires the original files to extract the missing peaks

## Inputs

- XCMS grouped xcmsSet object with peak detection, RT alignment, and peak grouping already done
- Raw LC-MS profile data (e.g., mzML/NetCDF CDF files) needed by fillPeaks to retrieve missing peaks
- ncGTW per-sample warping functions produced by ncGTW realignment of flagged misaligned feature groups

## Outputs

- Filled feature table (missing peaks retrieved) from baseline XCMS fillPeaks
- Realigned filled feature table from fillPeaksChromPar with ncGTW warping functions
- Per-feature coefficient of variation (CV) comparison written to cv_comparison.csv, with true positives marked as circles and false positives as crosses

## How to apply

Load the XCMS grouped output and the raw LC-MS profiles into R. First run XCMS fillPeaks with the original single warping function per sample to obtain the baseline filled table. Then substitute the ncGTW per-sample warping functions into XCMS via the replaced fillPeaksChromPar routine and re-run fillPeaks, which retrieves missing peaks at the ncGTW-specified locations. Compute per-feature coefficient of variation of intensity over samples with the ncGTW compCV function on both filled tables, then assemble a comparison table distinguishing true positives (circles) from false positives (crosses) and write it to cv_comparison.csv. The rationale is that ncGTW's compound-specific warping functions correct misalignments a single-warping-function alignment cannot, so peak-filling integrates more accurate intensities for true positives.

## Related tools

- **XCMS** (Provides fillPeaks and the fillPeaksChromPar routine that performs peak-filling after grouping using either its original warping functions or substituted ncGTW warping functions)
- **ncGTW** (Supplies per-sample realignment warping functions for substitution into fillPeaksChromPar and the compCV function used to compute per-feature coefficients of variation for quality comparison) — https://github.com/ChiungTingWu/ncGTW
- **R** (Execution environment (Bioconductor package ecosystem) in which the fillPeaks baseline/realigned runs and the compCV comparison are scripted)

## Examples

```
Rscript -e "xsF0 <- xcms::fillPeaks(xs); xsF1 <- xcms::fillPeaks(xs, param=replacedFillPeaksChromPar(ncGTWwarp)); ncGTW::compCV(xsF1)"
```

## Evaluation signals

- For true-positive misaligned features, ncGTW-guided peak-filling must consistently reduce the compCV coefficient of variation relative to the baseline XCMS fillPeaks run
- Reproduced CV values should match the reported behavior (Fig. 9): e.g., 0.369 -> 0.229 and 0.351 -> 0.119 for the two flagged features
- False positives of the misalignment detection must be distinguishable (crosses) from true positives (circles) in the cv_comparison.csv table
- Filled peaks in the realigned run must be retrieved at the ncGTW-specified locations, not the original single-warping-function locations
- The same compCV function must be applied identically to both filled tables so the CV comparison is a controlled comparison

## Limitations

- The false positive rate of ncGTW misalignment detection is higher than the theoretical threshold of 0.05 in the Rotterdam dataset, so the comparison table must screen out false positives via a post-alignment step
- ncGTW realignment can introduce peak shape distortion, which its post-processing module corrects; parameter settings are not exhaustively documented and users must consult the package defaults
- ncGTW assumes compound-specific RT drift varies over different m/z bins; features whose true warping functions match the single XCMS warping function will show no CV improvement from the substitution

## Evidence
