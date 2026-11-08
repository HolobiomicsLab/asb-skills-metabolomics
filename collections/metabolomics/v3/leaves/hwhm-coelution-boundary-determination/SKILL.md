---
name: hwhm-coelution-boundary-determination
description: Use when MRM ion chromatograms need automated peak-tip detection, left/right
  boundary tracing, and coelution flagging before quantitation — e.g. noisy, unevenly
  sampled, or isomeric (coeluted) peaks that must be resolved rather than merged.
when_to_use_negative:
- Input data have already been peak-detected and boundary-assigned by another tool
  (e.g. Skyline or MRMkit output) — this skill would re-detect tips and could disagree
  with the prior assignments.
- The chromatogram contains no coeluted isomers and the goal is only absolute quantitation
  from a standard curve, not boundary refinement.
- The data are unprocessed raw vendor files that have not yet been converted to mzML/.abf
  by msconvert.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MRMQuant
  role: Hosts the peak-tip tracing, HWHM computation, and coelution flagging pipeline;
    writes the resulting boundary and coelution information into its binary MAT-file
    structure for downstream quantitation.
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function used to find candidate peak tip locations in each
    ion chromatogram after dynamic signal smoothing.
- name: MATLAB
  role: Runtime environment in which findpeaks and the tracing/HWHM logic are executed;
    MRMQuant produces a binary MATLAB MAT-file.
- name: ProteoWizard
  role: Supplies msconvert to convert raw vendor MRM data into mzML or .abf files
    that MRMQuant can load before peak detection.
- name: msconvert
  role: ProteoWizard converter invoked to produce mzML or .abf sample files for the
    chromatograms being peak-detected.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:29d18f2b5d4169e2c0907c164fb3c1c9a144eac231f54af639c0dd73c52aceca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MATLAB
  - MRMQuant
  - ProteoWizard
  - findpeaks
  - msconvert
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# hwhm-coelution-boundary-determination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines peak boundaries and coelution status for each MRM ion chromatogram by tracing peak tips with MATLAB findpeaks, computing the half-width at half-maximum (HWHM), and flagging coelution when a neighboring peak intrudes within 2.5× the HWHM of the candidate peak.

## When to use

Use when MRM ion chromatograms need automated peak-tip detection, left/right boundary tracing, and coelution flagging before quantitation — e.g. noisy, unevenly sampled, or isomeric (coeluted) peaks that must be resolved rather than merged.

## When NOT to use

- Input data have already been peak-detected and boundary-assigned by another tool (e.g. Skyline or MRMkit output) — this skill would re-detect tips and could disagree with the prior assignments.
- The chromatogram contains no coeluted isomers and the goal is only absolute quantitation from a standard curve, not boundary refinement.
- The data are unprocessed raw vendor files that have not yet been converted to mzML/.abf by msconvert.

## Inputs

- MRM ion chromatograms extracted from MRM sample files (mzML or .abf format, converted via msconvert/ProteoWizard)
- MRMQuant method file containing targeted compound/transition information (RTs, precursor/product m/z)

## Outputs

- Per-transition peak-tip locations
- Per-transition HWHM (half-width at half-maximum) values
- Per-side coelution flags (left/right) and split boundaries at traced valleys
- MRMQuant binary MATLAB MAT-file structure consumed by later quantitation steps

## How to apply

For each ion chromatogram, apply dynamic signal smoothing and call MATLAB's built-in findpeaks to obtain candidate peak tips, keeping only tips that meet the minimum peak-width and minimum tip-distance thresholds. Trace each tip upward from the baseline until the tip signal declines past the 30% tip criterion, establishing the peak-tip region; at each traced boundary, apply the 5% tip criterion to decide whether the boundary marks a genuine valley between neighboring peaks or a shoulder of a coeluted peak. Compute the half-width-at-half-maximum (HWHM) of each candidate peak and flag left- or right-side coelution when a neighboring peak intrudes within 2.5× the HWHM bound on that side. Report the HWHM and coelution status for each side and write the per-transition tip locations, coelution flags, and split boundaries into the MRMQuant binary MAT-file structure used by later quantitation steps.

## Related tools

- **MRMQuant** (Hosts the peak-tip tracing, HWHM computation, and coelution flagging pipeline; writes the resulting boundary and coelution information into its binary MAT-file structure for downstream quantitation.) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function used to find candidate peak tip locations in each ion chromatogram after dynamic signal smoothing.)
- **MATLAB** (Runtime environment in which findpeaks and the tracing/HWHM logic are executed; MRMQuant produces a binary MATLAB MAT-file.)
- **ProteoWizard** (Supplies msconvert to convert raw vendor MRM data into mzML or .abf files that MRMQuant can load before peak detection.)
- **msconvert** (ProteoWizard converter invoked to produce mzML or .abf sample files for the chromatograms being peak-detected.)

## Examples

```
load mrqsample.mat; [tipLoc, hwhm, coelL, coelR] = MRMQuant.detectPeaks(chrom, 'MinPeakWidth', 5, 'MinPeakDistance', 20)
```

## Evaluation signals

- Peak tips and coelution splits must be internally consistent across technical replicates: the same transition should yield the same tip/valley assignments across the eight QC replicates.
- Replicate-to-replicate retention-time and STD variation for correctly handled peaks should be low (< 0.1), matching the reproducibility reported for MRMkit and Skyline in the comparison study.
- For chromatograms containing coeluted isomers (e.g. three PC 34:3 isomers with RTs at 4.72, 4.84, and 4.92 min), each isomer's abundance should be resolved separately rather than summed into one peak.
- Each traced boundary should extend no farther than 2.5× the computed HWHM from the peak tip, as the bounds are capped at 2.5 times the HWHM.
- Noisy peaks and peaks with unevenly sampled signals should be resolved (tip found and valley split identified) rather than merged or dropped, as occurred in the MRMkit/Skyline comparison.

## Limitations

- The paper reports no explicit numeric threshold for this stage itself; acceptance is the qualitative check that coeluted, noisy, and unevenly sampled peaks are resolved rather than merged or dropped.
- MRMkit and Skyline, which also use peak-boundary detection, exhibited problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals, indicating this problem is intrinsically difficult.
- Coelution at the targeted compound can still cause baseline estimation errors if the valley between neighbors is used to estimate the baseline, as seen in TargetLynx where high coelution in the right valley led to underestimation.

## Evidence

- [abstract] In MRMQuant, the MATLAB built-in function "findpeaks" is used to find possible peak tip locations: [evidence span withheld — anchor status: not_found]
- [abstract] traces the left- and right-hand bounds from the peak tip to the first local minimum, reports the half-width at half-maximum (HWHM) and coelution on each side, and restricts each bound to no larger than 2.5 times the HWHM: [evidence span withheld — anchor status: not_found]
