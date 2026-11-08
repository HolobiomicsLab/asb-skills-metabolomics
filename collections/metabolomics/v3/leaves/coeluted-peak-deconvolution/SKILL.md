---
name: coeluted-peak-deconvolution
description: Use when processing LC-MS/MS MRM chromatograms (mzML or vendor-converted
  raw data) where targeted compounds have coeluting isomers or neighbors on one side
  of the peak, and accurate per-isomer quantitation requires splitting those overlapping
  peaks before regression or abundance normalization.
when_to_use_negative:
- Input is already a quantitation table of peak areas and abundances — deconvolution
  is a pre-quantitation chromatogram step, not a post-hoc table operation.
- The chromatogram contains only one well-separated peak for the targeted compound
  with no neighboring tips within the 2.5x HWHM bound; automatic deconvolution adds
  overhead without changing the result.
- The tool being used cannot represent split/valley boundaries per isomer (e.g., TargetLynx-style
  valley-connecting baselines), so deconvolution output cannot be faithfully reproduced.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMQuant
  role: 'Primary tool that implements the deconvolution: finds peak tips with findpeaks,
    traces boundaries, flags coelution, and splits coeluted peaks before quantitation.'
  repo: https://github.com/kslynn128171/MRMQuant
- name: findpeaks
  role: MATLAB built-in function used inside MRMQuant to locate candidate peak tip
    locations in the smoothed chromatogram.
- name: MATLAB
  role: Runtime environment for the peak-detection and boundary-tracing logic; MRMQuant
    generates a binary MATLAB MAT-file consumed by later quantitation steps.
- name: ProteoWizard
  role: Used to convert vendor raw data to mzML via msconvert before MRMQuant consumes
    the chromatograms.
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:61d95ab20fb67f5b75a47bf660a7a0fa531390a8e3f11f9f4c6a9dd9c47e3414
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
  - MRMkit
  - Microsoft Excel
  - ProteoWizard
  - findpeaks
  - msconvert
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# coeluted-peak-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A quantitative-peak resolution procedure from the MRMQuant targeted-metabolomics paper: detect candidate peak tips in each MRM ion chromatogram with dynamic smoothing, trace valley boundaries with half-width-at-half-maximum (HWHM) and coelution flags, and deconvolute coeluted isomer peaks so their abundances are correctly attributed rather than merged into a single compound.

## When to use

Use when processing LC-MS/MS MRM chromatograms (mzML or vendor-converted raw data) where targeted compounds have coeluting isomers or neighbors on one side of the peak, and accurate per-isomer quantitation requires splitting those overlapping peaks before regression or abundance normalization.

## When NOT to use

- Input is already a quantitation table of peak areas and abundances — deconvolution is a pre-quantitation chromatogram step, not a post-hoc table operation.
- The chromatogram contains only one well-separated peak for the targeted compound with no neighboring tips within the 2.5x HWHM bound; automatic deconvolution adds overhead without changing the result.
- The tool being used cannot represent split/valley boundaries per isomer (e.g., TargetLynx-style valley-connecting baselines), so deconvolution output cannot be faithfully reproduced.

## Inputs

- MRM ion chromatograms from targeted metabolomics samples (e.g., mzML files converted from vendor raw data with msconvert in ProteoWizard, or binary MAT-file structures from prior MRMQuant steps)
- A method file with targeted compound information (precursor/product m/z, expected retention times, isomer groups) as loaded by MRMQuant
- Per-transition peak-tip parameters (minimum peak width, minimum tip distance, 30% and 5% tip criteria, 2.5x HWHM bound)

## Outputs

- Per-transition peak-tip locations and valley boundaries for each compound in each sample
- Left- and right-side coelution flags per candidate peak
- Split boundaries for coeluted isomer groups (e.g., the PC 34:3 RTs at 4.72, 4.84, and 4.92 min chromatogram)
- Deconvoluted peak abundances written to the MRMQuant binary MAT-file structure, later exported to a Microsoft Excel xlsx file

## How to apply

For each ion chromatogram, apply dynamic signal smoothing and use the MATLAB built-in `findpeaks` to locate candidate peak tips, filtering by minimum peak width and minimum tip-distance thresholds. Trace each tip upward from the baseline until the signal declines past the 30% tip criterion to establish the peak-tip region, and at each boundary apply a 5% tip criterion to decide whether the boundary is a genuine valley between neighboring peaks or the shoulder of a coeluted peak. Compute the HWHM for each candidate peak and flag left- or right-side coelution when a neighboring peak intrudes within the 2.5x HWHM bound on that side; this bound is the article's stated cap for how far each side of a peak may be traced. Deconvolute the coeluted region (e.g., by attributing signal to each isomer based on its own peak tip and valley boundaries) and write per-transition peak-tip locations, coelution flags, and split boundaries into the MRMQuant MAT-file structure consumed by downstream quantitation. The rationale is that tools like Skyline and TargetLynx either attribute all coeluted peaks to a single compound (e.g., the PC 34:3 three-isomer chromatogram) or underestimate abundance when coelution sits in the right valley, so explicit boundary tracing is needed to avoid over- or under-estimation.

## Related tools

- **MRMQuant** (Primary tool that implements the deconvolution: finds peak tips with findpeaks, traces boundaries, flags coelution, and splits coeluted peaks before quantitation.) — https://github.com/kslynn128171/MRMQuant
- **findpeaks** (MATLAB built-in function used inside MRMQuant to locate candidate peak tip locations in the smoothed chromatogram.)
- **MATLAB** (Runtime environment for the peak-detection and boundary-tracing logic; MRMQuant generates a binary MATLAB MAT-file consumed by later quantitation steps.)
- **ProteoWizard** (Used to convert vendor raw data to mzML via msconvert before MRMQuant consumes the chromatograms.)

## Examples

```
msconvert --mzML sample.raw; matlab -batch "MRMQuant('method.xlsx','sample.mzML','auto_deconvolution=true')"
```

## Evaluation signals

- For a chromatogram with known coeluting isomers (e.g., PC 34:3 RTs at 4.72, 4.84, and 4.92 min), each isomer should be assigned its own peak tip and abundance, not a single merged abundance — compare the per-isomer abundances against a manual split.
- Coelution flags should be symmetric with respect to the traced boundaries: if a neighbor is flagged as intruding on the left side, the corresponding split boundary must be present at the 5% tip criterion valley, not skipped.
- Across the eight technical-replicate phospholipid QC samples, the same transition should yield the same tip/valley assignments; MRMkit and Skyline's reported average STD <0.1 across replicates is the reference target for a correctly deconvoluted peak set.
- Every split boundary must lie within the 2.5x HWHM bound of the candidate peak — verify that no boundary is traced farther than 2.5x the half-width-at-half-maximum from the tip.
- Downstream batch-effect correction (BQC regression slope ~0.0, R2 ≈ 1.44 × 10–4) should remain attainable after deconvolution; if deconvolution is wrong, downstream correction may fail to flatten the BQC regression line.

## Limitations

- The 2.5x HWHM bound is a hard cap on how far each side of a peak may be traced; peaks with extremely wide or asymmetric coelution may still be misattributed if a neighbor lies beyond this bound.
- Deconvolution relies on the 30% and 5% tip criteria and on a minimum peak width/tip-distance threshold; chromatograms with noisy or unevenly sampled signals may produce incorrect valley assignments — the paper notes that MRMkit and Skyline had problematic quantitation for these cases, indicating this is a general difficulty.
- Manual inspection of the visualized peaks is still allowed when automatic deconvolution fails, so the result is not guaranteed to be fully automatic.
- MRMQuant is a Windows-targeted MATLAB program; the article notes it produces a binary MATLAB MAT-file, so a Linux/Python pipeline cannot consume the deconvolution output without that runtime.

## Evidence

- [abstract] findpeaks and boundary tracing: [evidence span withheld — anchor status: not_found]
- [full_text] task card finding (workflow details): [evidence span withheld — anchor status: not_found]
- [full_text] task card validation signal: [evidence span withheld — anchor status: not_found]
