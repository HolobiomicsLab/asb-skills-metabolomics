---
name: missing-peak-apex-estimation
description: Use when XCMS peak-filling left a flagged feature undetected in one or
  more samples and the ncGTW realignment warping functions must supply an apex position
  and rigidly aligned peak boundaries for those missing peaks before results are handed
  back to XCMS.
when_to_use_negative:
- Feature apices are already co-aligned across samples and peak shapes (width/height
  profiles) are preserved — no distortion correction is needed.
- Misaligned features have not yet been detected/flagged; run the ncGTW misalignment
  detection module first.
- No XCMS peak table or peak start/end/width information exists, so there is no apex
  constraint or median peak width to align missing peaks against.
tools:
- name: ncGTW
  role: 'Provides the post-processing module: adjustRT() regenerates RT warping functions
    and ncGTWpeaks relocates peaks (including apex estimation for missing peaks on
    the virtual reference) before results are sent back to xcms'
  repo: https://github.com/ChiungTingWu/ncGTW
- name: XCMS
  role: Supplies the peak start/end points, peak widths, and peak-filling procedure
    whose missing peaks this skill recovers; consumes the corrected warping functions
    and relocated peaks for regrouping and peak-filling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaa037
    title: Targeted realignment of LC-MS profiles by neighbor-wise compound-specific
      graphical time warping with misalignment detection
  source_content_hash: sha256:2a034b3feb22adcf8ec9040c56ddfbd744b17df271e31931e0da4a8e1b5d8af1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ChiungTingWu/ncGTW
  tools:
  - R
  - XCMS
  - ncGTW
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# missing-peak-apex-estimation

## Summary

Recovers apex positions and peak boundaries for features that XCMS peak detection/peak-filling missed in some samples, using the ncGTW realignment output: the apex is located on the virtual reference profile aggregated from the other samples and mapped through the per-sample warping function, so the whole missing peak can be aligned and returned to XCMS.

## When to use

Use when XCMS peak-filling left a flagged feature undetected in one or more samples and the ncGTW realignment warping functions must supply an apex position and rigidly aligned peak boundaries for those missing peaks before results are handed back to XCMS.

## When NOT to use

- Feature apices are already co-aligned across samples and peak shapes (width/height profiles) are preserved — no distortion correction is needed.
- Misaligned features have not yet been detected/flagged; run the ncGTW misalignment detection module first.
- No XCMS peak table or peak start/end/width information exists, so there is no apex constraint or median peak width to align missing peaks against.

## Inputs

- ncGTW realignment output (per-feature per-sample warping functions and aligned profiles, e.g. an ncGTW object)
- xcmsLargeWin XCMS object with corrected RT, peak start/end apex points, and peak widths per feature
- LC-MS profile data (m/z bins) for features flagged as misaligned by the ncGTW detection module

## Outputs

- Corrected RT warping functions replacing xcmsLargeWin@rt$corrected (via adjustRT())
- ncGTWpeaks output: relocated peak positions with apex-aligned, rigidly shifted peak boundaries
- Per-feature intensity table across samples, including peaks recovered for samples where XCMS detection missed the peak

## How to apply

Load the ncGTW realigned feature profiles together with the XCMS-provided peak start/end points and peak widths for each flagged feature. For samples where XCMS did find the peak, restrict the realignment warping function to the apex inside the peak's start-end boundaries and shift all remaining in-peak points rigidly by the same apex displacement, preventing the many-to-one/one-to-many mappings that broaden or narrow peaks. For samples with missing peaks undetected by XCMS, locate the apex on the virtual reference aggregated from the other samples, assign the corresponding warping-function point as that sample's apex, and align the whole missing peak using the median of the peak widths across samples. Assemble the corrected profiles into a per-feature intensity table across samples, apply adjustRT() so the new warping functions replace xcmsLargeWin@rt$corrected, and use ncGTWpeaks to hand relocated peak positions back to XCMS for more accurate grouping and peak-filling.

## Related tools

- **ncGTW** (Provides the post-processing module: adjustRT() regenerates RT warping functions and ncGTWpeaks relocates peaks (including apex estimation for missing peaks on the virtual reference) before results are sent back to xcms) — https://github.com/ChiungTingWu/ncGTW
- **XCMS** (Supplies the peak start/end points, peak widths, and peak-filling procedure whose missing peaks this skill recovers; consumes the corrected warping functions and relocated peaks for regrouping and peak-filling)

## Examples

```
devtools::install_github("ChiungTingWu/ncGTW")
```

## Evaluation signals

- Within each corrected feature, the apices remain co-aligned across samples (the Fig. 5c-to-5d correction criterion).
- Peak shape (width/height profile) is preserved relative to the pre-correction realigned profiles — no broadening or narrowing introduced.
- Each in-peak point shifts by exactly the apex displacement: no many-to-one or one-to-many RT mappings appear in the corrected warping functions.
- Recovered missing peaks, aligned with the median peak width, land on the virtual reference aggregated from the other samples.
- Downstream XCMS regrouping and peak-filling on the realigned data yields more accurate grouping than the uncorrected run.

## Limitations

- ncGTW parameter settings are not exhaustively listed or optimized in the text; users must consult the package for defaults, and suboptimal settings can leave peak distortion.
- False positives from the misalignment detection stage are deliberately accepted first and must be screened out later via correlation-coefficient and overlapping-area scores in the post-alignment step.
- Apex-restricted rigid shifting assumes the whole peak moves rigidly with the apex; peaks with strongly nonlinear internal distortion are not reshaped by this correction.

## Evidence

- [methods] Peak shape distortion is then efficiently corrected by the post-processing module of ncGTW package.: [evidence span withheld — anchor status: not_found]
