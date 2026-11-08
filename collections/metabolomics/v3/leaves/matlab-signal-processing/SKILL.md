---
name: matlab-signal-processing
description: Use when processing high-throughput LC-MS/MS MRM ion chromatograms (mzML
  or MRMQuant MAT-file samples) where you need to locate targeted-compound peak tips
  and resolve coeluted isomers before absolute quantitation.
when_to_use_negative:
- Quantitation is already available and you only need batch-effect correction of BQC
  slopes — use MRMQuant's batch-effect correction instead of re-running peak detection.
- Input is a raw vendor file (e.g. Waters or Agilent) that has not yet been converted
  with ProteoWizard msconvert — run format conversion before invoking MATLAB signal
  processing.
- Manual inspection/correction of already-quantitated peaks is required — MRMQuant's
  visualized inspection step is a separate post-detection stage.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: Implements the dynamic smoothing, 30% / 5% tip criteria, HWHM computation,
    and 2.5× HWHM coelution-split logic over MRM chromatograms, persisting results
    into its MAT-file structure.
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function called by MRMQuant to identify possible peak-tip
    locations in each smoothed ion chromatogram.
- name: MATLAB
  role: Runtime environment in which findpeaks and MRMQuant's peak-detection/quantitation
    routines execute and the MAT-file is generated.
- name: msconvert
  role: ProteoWizard tool used upstream to convert raw vendor MRM files into mzML
    or MRM sample files that feed the MATLAB signal processing.
- name: ProteoWizard
  role: Suite providing msconvert for raw-to-mzML/sample-file conversion prior to
    MATLAB peak detection.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:12becc5b8980b53b30c74da843ba5450655937917b755726e6fe958d8cf86c0b
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# matlab-signal-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Peak-tip detection and boundary tracing on MRM ion chromatograms using MATLAB's built-in findpeaks plus MRMQuant's dynamic smoothing, HWHM-based coelution flagging, and 2.5×HWHM split-bound restriction.

## When to use

Use when processing high-throughput LC-MS/MS MRM ion chromatograms (mzML or MRMQuant MAT-file samples) where you need to locate targeted-compound peak tips and resolve coeluted isomers before absolute quantitation.

## When NOT to use

- Quantitation is already available and you only need batch-effect correction of BQC slopes — use MRMQuant's batch-effect correction instead of re-running peak detection.
- Input is a raw vendor file (e.g. Waters or Agilent) that has not yet been converted with ProteoWizard msconvert — run format conversion before invoking MATLAB signal processing.
- Manual inspection/correction of already-quantitated peaks is required — MRMQuant's visualized inspection step is a separate post-detection stage.

## Inputs

- MRM ion chromatograms (mzML or MRM sample files converted via ProteoWizard msconvert, or pre-loaded MRMQuant MAT-file samples)
- Method file with targeted compound RTs and transition information
- User-configurable minimum peak width and minimum tip-distance thresholds for findpeaks

## Outputs

- Per-transition peak-tip locations (RTs) stored in the MRMQuant binary MAT-file structure
- Per-side coelution flags (left/right) indicating neighboring peaks within the 2.5× HWHM bound
- Split boundaries and half-width-at-half-maximum (HWHM) values for each candidate peak

## How to apply

For each ion chromatogram, apply MRMQuant's dynamic signal smoothing and call candidate peak tips with the MATLAB built-in function findpeaks, keeping tips above the minimum peak width and minimum tip-distance thresholds. Trace each candidate tip upward from the baseline until the tip signal declines past the 30% tip criterion to establish the peak-tip region. At each traced boundary, apply the 5% tip criterion to decide whether the boundary marks a genuine valley between neighboring peaks or a shoulder of a coeluted peak. Compute the half-width-at-half-maximum (HWHM) for each candidate peak and flag left- or right-side coelution when a neighboring peak intrudes within the 2.5× HWHM bound on that side; trace the left- and right-hand bounds from the peak tip to the first local minimum, restricting each bound to no larger than 2.5 times the HWHM. Write the per-transition peak-tip locations, coelution flags, and split boundaries into the MRMQuant binary MAT-file structure consumed by later quantitation steps.

## Related tools

- **MRMQuant** (Implements the dynamic smoothing, 30% / 5% tip criteria, HWHM computation, and 2.5× HWHM coelution-split logic over MRM chromatograms, persisting results into its MAT-file structure.) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function called by MRMQuant to identify possible peak-tip locations in each smoothed ion chromatogram.)
- **MATLAB** (Runtime environment in which findpeaks and MRMQuant's peak-detection/quantitation routines execute and the MAT-file is generated.)
- **msconvert** (ProteoWizard tool used upstream to convert raw vendor MRM files into mzML or MRM sample files that feed the MATLAB signal processing.)
- **ProteoWizard** (Suite providing msconvert for raw-to-mzML/sample-file conversion prior to MATLAB peak detection.)

## Examples

```
mrunquant('sample_files', 'method.meth') — MRMQuant invokes MATLAB findpeaks internally; or convert first: msconvert raw.wiff --mzML
```

## Evaluation signals

- Across the eight technical QC replicates of a phospholipid sample, the same transition must yield identical tip/valley assignments; observed average STD and RT differences should stay below 0.1 as reported for well-resolved peaks.
- Every split boundary must lie within 2.5× the peak's half-width-at-half-maximum (HWHM); boundaries exceeding this bound indicate a tracing failure.
- Coeluted isomers (e.g. the three PC 34:3 peaks at RTs 4.72, 4.84, and 4.92 min) must be attributed to distinct peaks rather than merged — all three RT values should appear as separate tips.
- Peak tips must exceed both the minimum peak width and minimum tip-distance thresholds supplied to findpeaks; tips below these thresholds are discarded.
- Resulting peak tips, coelution flags, and split boundaries must be present in the binary MAT-file structure and match the values later loaded into MRMQuant's quantitation stage.

## Limitations

- MRMkit and Skyline also achieve low (<0.1) replicate STD/RT differences but had problematic quantitation of coeluted peaks, noisy peaks, and peaks with unevenly sampled signals; MRMQuant addresses this via dynamic smoothing and deconvolution, but very low-quality signals may still fail.
- No numeric threshold is stated for the peak-detection stage itself — acceptance is a qualitative check that coeluted, noisy, and unevenly sampled peaks are resolved rather than merged or dropped.
- The method was validated only on the phospholipid QC replicates and bile-acid/phospholipid data sets from the human aging study; behaviour on other analyte classes or instruments is unverified.
- Be sure to use the latest MRMQuant version (currently v2.7); older versions may behave differently.

## Evidence

- [methods] findpeaks-based tip detection with 2.5× HWHM bound restriction: [evidence span withheld — anchor status: not_found]
- [results] coelution flagging within 2.5× HWHM: [evidence span withheld — anchor status: not_found]
