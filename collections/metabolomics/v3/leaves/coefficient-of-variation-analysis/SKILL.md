---
name: coefficient-of-variation-analysis
description: Use when two peak-filling strategies (e.g., XCMS default fillPeaks vs.
  ncGTW-realigned fillPeaks) must be compared on feature intensity consistency, to
  verify that realignment reduces CV for detected misaligned features.
when_to_use_negative:
- The feature set contains only true negatives (well-aligned features) — ncGTW-guided
  filling is expected to help flagged misaligned features, and CV comparison is not
  the right screen for general feature quality.
- Only a final feature table is available without raw LC-MS profiles — fillPeaks cannot
  retrieve missing peaks from integrated peaks alone.
- The alignment already used compound-specific per-sample warping functions — a single-warping-function
  XCMS baseline comparison is not applicable.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ncGTW
  role: Provides the compCV function used to compute per-feature CV of filled peak
    intensities, and the replaced fillPeaksChromPar routine for injecting ncGTW warping
    functions into XCMS fillPeaks
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Runs fillPeaks twice — baseline with its own single warping function and again
    with ncGTW warping functions — to produce the two filled feature tables being
    compared
- name: R
  role: Language environment (Bioconductor) in which the XCMS/ncGTW pipeline and CV
    comparison are executed
  repo: https://github.com/ChiungTingWu/ncGTW
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:bf0191caa61c31ba9288a52adcb8ad073b4d086fe5c73c3ea7802e992a7fa9e3
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# coefficient-of-variation-analysis

## Summary

Quantifies peak-filling quality for misaligned LC-MS features by computing the per-feature coefficient of variation (CV) of filled peak intensities across samples, comparing an XCMS single-warping-function baseline against ncGTW realignment warping functions via the ncGTW compCV function.

## When to use

Use when two peak-filling strategies (e.g., XCMS default fillPeaks vs. ncGTW-realigned fillPeaks) must be compared on feature intensity consistency, to verify that realignment reduces CV for detected misaligned features.

## When NOT to use

- The feature set contains only true negatives (well-aligned features) — ncGTW-guided filling is expected to help flagged misaligned features, and CV comparison is not the right screen for general feature quality.
- Only a final feature table is available without raw LC-MS profiles — fillPeaks cannot retrieve missing peaks from integrated peaks alone.
- The alignment already used compound-specific per-sample warping functions — a single-warping-function XCMS baseline comparison is not applicable.

## Inputs

- XCMS grouped feature output (xcmsSet/grouped object)
- ncGTW per-sample warping functions for detected misaligned features (from ncGTW realignment)
- Raw LC-MS profile data (required for fillPeaks peak retrieval)
- Misalignment detection labels distinguishing true positives from false positives

## Outputs

- cv_comparison.csv — per-feature CV under XCMS baseline filling vs. ncGTW-realigned filling
- True-positive/false-positive annotation (circles vs. crosses) for the compared features

## How to apply

Load the XCMS grouped output, the ncGTW per-sample warping functions from the prior realignment task, and the raw LC-MS profiles. First run XCMS fillPeaks with the original single XCMS warping functions to produce the baseline filled feature table. Then substitute the ncGTW warping functions into XCMS via the replaced fillPeaksChromPar routine and re-run fillPeaks so missing peaks are retrieved at the ncGTW-specified locations. For both filled tables, compute the per-feature coefficient of variation of intensity over the samples using the ncGTW compCV function. Assemble a comparison table that marks true positives (circles) versus false positives (crosses) of the misalignment detection, and expect ncGTW-guided filling to consistently lower CV for true-positive features — e.g., 0.369→0.229 and 0.351→0.119 for the two flagged features — mirroring the behavior reported in Fig. 9 for the Rotterdam and MESA datasets.

## Related tools

- **ncGTW** (Provides the compCV function used to compute per-feature CV of filled peak intensities, and the replaced fillPeaksChromPar routine for injecting ncGTW warping functions into XCMS fillPeaks) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Runs fillPeaks twice — baseline with its own single warping function and again with ncGTW warping functions — to produce the two filled feature tables being compared)
- **R** (Language environment (Bioconductor) in which the XCMS/ncGTW pipeline and CV comparison are executed) — https://github.com/ChiungTingWu/ncGTW

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW")
```

## Evaluation signals

- CV computed with ncGTW compCV must decrease after ncGTW-guided filling for true-positive features (direction check: 0.369→0.229 and 0.351→0.119 in the reported example).
- The same sample set and per-feature scope must be used when running compCV on both filled tables so CVs are directly comparable.
- The ncGTW-substituted fillPeaks run must retrieve the previously missing peaks at the ncGTW-specified locations (filled table completeness check).
- The comparison table must annotate detection outcomes consistently (true positives as circles, false positives as crosses), matching the Fig. 9 behavior on the Rotterdam and MESA datasets.
- cv_comparison.csv must contain paired CV columns (before vs. after realignment) for every flagged feature.

## Limitations

- ncGTW's misalignment detection admits false positives; the post-alignment screening step (correlation coefficient and overlapping area scores) is needed to identify true positives, and CV reduction should not be expected for false positives.
- CV improvement is demonstrated for features whose true warping functions differ per sample; features whose misalignment is correctable by tuning a single warping function will not show meaningful gains.
- Exact screening criteria for false positives and exhaustive ncGTW parameter settings are not detailed in the article; consult the package defaults.

## Evidence

- [abstract] plug-in role of ncGTW in XCMS pipeline: "The ncGTW software tool is developed currently as a plug-in to detect and realign misaligned features present in standard XCMS output."
- [intro] realigned data used for peak-filling: "ncGTW can detect the misaligned feature groups from XCMS and realign them. After that, XCMS can use the realigned data from XCMS for more accurate grouping and peak-filling."
- [methods] peak-filling retrieves missing peaks: "those missing peaks may be accurately retrieved by the peak-filling procedure in XCMS"
- [methods] true/false positive screening of detected features: "via a post-alignment step we use these two performance scores to screen out true positives for further analysis."
- [readme] ncGTW as R plug-in for XCMS: "Currently, ncGTW is implemented in a R-package as a plug-in for XCMS. That is, ncGTW can detect the misaligned feature groups from XCMS and realign them."
