---
name: retention-time-warping-function-substitution
description: Use when XCMS grouping has flagged features as misaligned and ncGTW has
  produced per-sample warping functions, and you want more accurate peak-filling (integration
  of missing peaks) than the default single-warping-function XCMS alignment provides.
when_to_use_negative:
- The features are not flagged as misaligned by the ncGTW misalignment detection module
  (57 flagged in the paper's example); substituting warping functions for well-aligned
  features is unnecessary
- You lack the ncGTW warping functions because misalignment detection/realignment
  has not yet been run; only the XCMS single-warping-function baseline can be produced
- The goal is general preprocessing of a new dataset from scratch (peak detection,
  RT alignment, grouping) rather than targeted repair of a specific XCMS output
tools:
- name: ncGTW
  role: Supplies the per-sample warping functions for misaligned features and the
    compCV function used to evaluate peak-filling quality; includes the fillPeaksChromPar
    replacement for substituting warping functions into XCMS
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Performs peak detection, RT alignment, grouping, and peak-filling; its fillPeaks
    routine is re-run with substituted ncGTW warping functions
- name: R
  role: Execution environment; ncGTW is an R/Bioconductor package installed as a plug-in
    for XCMS
- name: Bioconductor
  role: Distribution channel for the ncGTW R package
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:2ca02b37779b3618b6b1741ee5e50b2d03731ede81c4a3860cb2b442aa2c9ec9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - Bioconductor
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# retention-time-warping-function-substitution

## Summary

Substitutes ncGTW's per-sample, compound-specific RT warping functions into XCMS's peak-filling routine (via the replaced fillPeaksChromPar routine) so missing peaks are retrieved at ncGTW-specified locations instead of using XCMS's single warping function per sample, improving peak-filling quality for detected misaligned features.

## When to use

Use when XCMS grouping has flagged features as misaligned and ncGTW has produced per-sample warping functions, and you want more accurate peak-filling (integration of missing peaks) than the default single-warping-function XCMS alignment provides.

## When NOT to use

- The features are not flagged as misaligned by the ncGTW misalignment detection module (57 flagged in the paper's example); substituting warping functions for well-aligned features is unnecessary
- You lack the ncGTW warping functions because misalignment detection/realignment has not yet been run; only the XCMS single-warping-function baseline can be produced
- The goal is general preprocessing of a new dataset from scratch (peak detection, RT alignment, grouping) rather than targeted repair of a specific XCMS output

## Inputs

- XCMS grouped feature output (xcmsSet object with grouping/alignment results)
- ncGTW per-sample warping functions for the detected misaligned feature groups
- Raw LC-MS profile data (centroided profiles underlying the XCMS object)

## Outputs

- Baseline filled feature table from XCMS fillPeaks with original warping functions
- Realigned filled feature table from fillPeaks with ncGTW warping functions substituted via fillPeaksChromPar
- Per-feature coefficient of variation (CV) comparison table (e.g., cv_comparison.csv) distinguishing true positives from false positives

## How to apply

Load the XCMS grouped output, the ncGTW per-sample warping functions from the earlier realignment task, and the raw LC-MS profiles into R. First run XCMS fillPeaks with the original XCMS warping functions to obtain the baseline filled feature table. Then substitute the ncGTW warping functions into XCMS via the replaced fillPeaksChromPar routine and re-run fillPeaks so that missing peaks are retrieved at the ncGTW-specified RT locations. Quantify the improvement with the ncGTW compCV function, computing per-feature coefficient of variation (CV) of intensity over samples for both filled tables, and assemble a comparison table distinguishing true positives (circles) from false positives (crosses) of the misalignment detection. Validation: ncGTW-guided peak-filling should consistently reduce CV relative to XCMS-only peak-filling for true-positive features, as reported for the Rotterdam and MESA datasets (e.g., CV 0.369 to 0.229 and 0.351 to 0.119 for two misaligned features).

## Related tools

- **ncGTW** (Supplies the per-sample warping functions for misaligned features and the compCV function used to evaluate peak-filling quality; includes the fillPeaksChromPar replacement for substituting warping functions into XCMS) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Performs peak detection, RT alignment, grouping, and peak-filling; its fillPeaks routine is re-run with substituted ncGTW warping functions)
- **R** (Execution environment; ncGTW is an R/Bioconductor package installed as a plug-in for XCMS)
- **Bioconductor** (Distribution channel for the ncGTW R package)

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW"); xcmsFill <- fillPeaks(xs, chromPar=replacedFillPeaksChromPar(ncGTWWarp))
```

## Evaluation signals

- Per-feature CV of intensity over samples (via ncGTW compCV) must be lower with ncGTW warping functions than with XCMS's original warping functions for true-positive misaligned features (paper reports 0.369->0.229 and 0.351->0.119)
- Missing peaks must be integrated at the ncGTW-specified locations after fillPeaksChromPar substitution, i.e., previously missing peaks are retrieved in the realigned filled table
- True positives (circles) and false positives (crosses) of the misalignment detection are distinguishable in the CV comparison table
- Behavior must match the reported Fig. 9 pattern for the Rotterdam and MESA datasets
- False positives of misalignment detection should not show a consistent CV reduction, indicating the improvement is specific to genuinely misaligned features

## Limitations

- ncGTW can cause peak distortion; parameter settings may need adjustment to reduce its likelihood, and a post-processing module corrects peak shape distortion
- The misalignment detection has an elevated false positive rate (e.g., higher than the theoretical 0.05 threshold in the Rotterdam dataset), so CV improvements should be screened via correlation coefficient and overlapping area scores to confirm true positives
- Exact implementation details of the fillPeaksChromPar substitution and parameter defaults must be consulted in the ncGTW package; the paper does not list all parameter settings or software versions

## Evidence
