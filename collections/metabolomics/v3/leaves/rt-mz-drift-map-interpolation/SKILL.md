---
name: rt-mz-drift-map-interpolation
description: Use when a public or parameter-free LC/MS dataset lacks experimental
  QC injections but raw mzML-derived MS1 ion-feature matrices still show run-order/batch
  intensity drift that must be normalized before downstream modeling.
when_to_use_negative:
- The input is already a normalized feature table or the drift correction was already
  applied — re-running would double-correct intensities
- Experimental QC samples with standardized injections are actually available, in
  which case conventional QC-based drift estimation can be used directly
- The analysis requires MS/MS-level de-isotoping, annotation, or compound identification
  — mzLearn results are at the ion MS1 level and do not perform these steps
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: mzLearn
  role: Iteratively clusters run-order-consecutive samples to define synthetic QCs,
    builds per-feature intensity drift maps via rt/m/z interpolation, and normalizes
    the feature matrix
  repo: github.com/ReviveMed/mzEmbed
- name: mzEmbed
  role: Consumes the normalized 2,736-feature matrix downstream for VAE pretraining;
    the drift-map skill prepares its input matrices
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:82159be0cee1afdc56a55e431f99b1195df81ad05714f5acd730485c233098ee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# rt-mz-drift-map-interpolation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs synthetic QC samples and per-feature intensity drift maps in mzLearn when experimental QC samples are absent, by clustering run-order-consecutive samples and interpolating drift between neighboring pins in rt and m/z space.

## When to use

Use when a public or parameter-free LC/MS dataset lacks experimental QC injections but raw mzML-derived MS1 ion-feature matrices still show run-order/batch intensity drift that must be normalized before downstream modeling.

## When NOT to use

- The input is already a normalized feature table or the drift correction was already applied — re-running would double-correct intensities
- Experimental QC samples with standardized injections are actually available, in which case conventional QC-based drift estimation can be used directly
- The analysis requires MS/MS-level de-isotoping, annotation, or compound identification — mzLearn results are at the ion MS1 level and do not perform these steps

## Inputs

- MS1 ion-feature intensity matrix produced by mzLearn's signal detection and alignment of mzML input files
- Run order (injection order) of samples to define run-order-consecutive neighborhoods
- Feature rt and m/z coordinates (median rt and m/z values per detected signal) for spatial interpolation of drift maps

## Outputs

- drift_maps.csv — per-feature intensity drift maps
- normalized_feature_matrix.csv — drift-corrected two-dimensional table of detected features (median rt, median m/z, normalized intensities across samples)
- Synthetic QC samples defined as per-cluster average intensities

## How to apply

Start from the MS1 ion-feature intensity matrix produced by mzLearn's signal detection and alignment of mzML files. Iteratively cluster only run-order-consecutive samples using low-dimensional representations of their intensity profiles, letting the deterministic, non-parametric routine learn internal thresholds from the data rather than setting user parameters. Define each synthetic QC sample as the per-cluster average intensity, replacing the absent experimental QCs. Treat the synthetic QC intensities as pins and interpolate between neighboring pins in rt and m/z space to construct a per-feature intensity drift map, then apply the map to normalize feature intensities across the cohort. Save the per-feature drift maps to drift_maps.csv and the normalized matrix to normalized_feature_matrix.csv.

## Related tools

- **mzLearn** (Iteratively clusters run-order-consecutive samples to define synthetic QCs, builds per-feature intensity drift maps via rt/m/z interpolation, and normalizes the feature matrix) — github.com/ReviveMed/mzEmbed
- **mzEmbed** (Consumes the normalized 2,736-feature matrix downstream for VAE pretraining; the drift-map skill prepares its input matrices) — github.com/ReviveMed/mzEmbed

## Evaluation signals

- The normalized feature matrix shows reduced relative standard deviation (RSD) across samples compared with the unnormalized matrix (the paper validates this module only via downstream replicate-variability reduction)
- drift_maps.csv contains a per-feature drift map entry for each detected signal defined by median rt and median m/z
- Each synthetic QC equals the average intensity of one run-order-consecutive cluster, not of the full cohort
- Normalized matrix retains the same feature rows as the input MS1 matrix (e.g., the robust 2,736-feature set used for mzEmbed pretraining)

## Limitations

- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, which is why synthetic QCs must be reconstructed
- Iterative learning increases computational time, up to 2.5 days on a single machine for large cohorts; parallel processing on multiple compute nodes is not yet implemented
- mzLearn results are at the ion MS1 level and do not perform de-isotoping or MS/MS identification

## Evidence

- [other] synthetic QC definition and interpolation: [evidence span withheld — anchor status: not_found]
- [readme] README drift correction description: [evidence span withheld — anchor status: external_doc]
