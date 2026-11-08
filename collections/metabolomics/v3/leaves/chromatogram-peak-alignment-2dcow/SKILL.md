---
name: chromatogram-peak-alignment-2dcow
description: Use when a smoothed, baseline-corrected two-dimensional chromatogram
  (2D-TIC) must be warped pixel-wise onto a reference chromatogram of the same dataset
  to remove retention-time shifts before multivariate analysis.
when_to_use_negative:
- Input chromatograms are raw NetCDF imports that have not yet been baseline-corrected
  (asymmetric least squares) and smoothed — preprocessing precedes peak alignment
  in the RGCxGC workflow
- A whole batch of chromatograms must be aligned together — use the batch_2DCOW interface
  (reference, sample_chroms, segments, max_warp, add_ref) rather than single-sample
  twod_cow
- The data are one-dimensional chromatograms or feature tables rather than folded
  two-dimensional GC×GC 2D-TIC chromatograms
tools:
- name: RGCxGC
  role: 'Provides the twod_cow function implementing 2D-COW chromatogram alignment
    (parameters: reference, sample_chroms, segments, max_warp, add_ref)'
  repo: https://github.com/DanielQuiroz97/RGCxGC
- name: ptw
  role: Supplies the base functions with1 and with2 coupled into RGCxGC's 2D-COW warping
    implementation
- name: R
  role: Runtime environment in which the RGCxGC vignette executes the alignment and
    serializes the result with saveRDS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.microc.2020.104830
    title: 'RGCxGC toolbox: An R-package for data processing in comprehensive two-dimensional
      gas chromatography-mass spectrometry'
  source_content_hash: sha256:21028f893877d35c105c69a126b6d80af32c89773ec694bf04dcf188d6ec34e5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/DanielQuiroz97/RGCxGC
  repo_url: https://github.com/DanielQuiroz97/RGCxGC
  tools:
  - R
  - RGCxGC
  - ptw
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chromatogram-peak-alignment-2dcow

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns a single preprocessed GC×GC 2D-TIC chromatogram to a reference chromatogram by pixel-level two-dimensional correlation optimized warping (2D-COW) via the twod_cow function of the RGCxGC package, using per-dimension segment counts and warp limits.

## When to use

Use when a smoothed, baseline-corrected two-dimensional chromatogram (2D-TIC) must be warped pixel-wise onto a reference chromatogram of the same dataset to remove retention-time shifts before multivariate analysis.

## When NOT to use

- Input chromatograms are raw NetCDF imports that have not yet been baseline-corrected (asymmetric least squares) and smoothed — preprocessing precedes peak alignment in the RGCxGC workflow
- A whole batch of chromatograms must be aligned together — use the batch_2DCOW interface (reference, sample_chroms, segments, max_warp, add_ref) rather than single-sample twod_cow
- The data are one-dimensional chromatograms or feature tables rather than folded two-dimensional GC×GC 2D-TIC chromatograms

## Inputs

- Reference two-dimensional chromatogram object (e.g. MTBL09_sm2, GC×GC 2D-TIC with baseline correction and Whittaker smoothing applied)
- Sample two-dimensional chromatogram object to be warped (e.g. MTBLS08_sm2, preprocessed identically to the reference)
- Integer vector segments = c(10, 40): number of segments in the first and second dimensions
- Integer vector max_warp = c(1, 8): maximum allowed warp points in the first and second dimensions

## Outputs

- Aligned two-dimensional chromatogram object of the same dimensionality as the input sample, warped against the reference
- Serialized RDS file (e.g. aligned_08GB.rds) written with saveRDS

## How to apply

Load both chromatograms as RGCxGC two-dimensional chromatogram objects; in the vignette the smoothed 08GB sample (MTBLS08_sm2) is aligned to the equivalently preprocessed 09GB reference (MTBL09_sm2), each preprocessed with asymmetric least squares baseline correction and Whittaker (quadratic-penalty) smoothing. Call twod_cow from RGCxGC, passing the reference chromatogram and the sample chromatogram, with segments = c(10, 40) giving the number of segments in the first and second chromatographic dimensions and max_warp = c(1, 8) giving the allowed warp points per dimension. The 2D-COW algorithm internally relies on the base functions with1 and with2 coupled from the ptw package to compute the optimal warp of the sample onto the reference. Persist the returned aligned chromatogram object with saveRDS (e.g. aligned_08GB.rds); the warping limits trade alignment flexibility against peak distortion, so keep max_warp modest in each dimension.

## Related tools

- **RGCxGC** (Provides the twod_cow function implementing 2D-COW chromatogram alignment (parameters: reference, sample_chroms, segments, max_warp, add_ref)) — https://github.com/DanielQuiroz97/RGCxGC
- **ptw** (Supplies the base functions with1 and with2 coupled into RGCxGC's 2D-COW warping implementation)
- **R** (Runtime environment in which the RGCxGC vignette executes the alignment and serializes the result with saveRDS)

## Examples

```
aligned_08GB <- twod_cow(reference = MTBL09_sm2, sample_chroms = MTBLS08_sm2, segments = c(10, 40), max_warp = c(1, 8))
```

## Evaluation signals

- The output file aligned_08GB.rds exists after execution
- When loaded, aligned_08GB.rds contains a two-dimensional chromatogram object of the same dimensionality as the input sample chromatogram
- The aligned chromatogram has been warped against the 09GB (MTBL09_sm2) reference, i.e. the warping was computed relative to that specific reference chromatogram
- The call used the vignette's segmentation and warping limits: segments = c(10, 40) and max_warp = c(1, 8)
- Sample and reference were preprocessed identically (asymmetric least squares baseline correction plus quadratic-penalty/Whittaker smoothing) before warping

## Limitations

- The warp limits max_warp = c(1, 8) constrain how far peaks can move per dimension; shifts exceeding these limits in either the first or second chromatographic dimension will not be corrected
- Alignment quality depends on the reference choice — the vignette warps the 08GB sample onto the 09GB reference within the MTBLS579 dataset, and a poor reference yields a misleadingly warped chromatogram
- 2D-COW here addresses peak alignment only; it does not substitute for the other preprocessing steps (noise smoothing and baseline correction) required before multivariate analysis

## Evidence

    ]"
