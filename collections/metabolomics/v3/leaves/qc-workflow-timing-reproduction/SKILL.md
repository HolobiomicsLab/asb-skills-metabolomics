---
name: qc-workflow-timing-reproduction
description: Use when you need to reconstruct or benchmark a lipid retention-time
  QC model from a peak/feature table (e.g. LipiDex 2 demo dataset) and verify that
  model fitting completes within a target runtime such as 10 seconds.
when_to_use_negative:
- Input is a raw Thermo .raw or .mzML file rather than a post-identification lipid
  feature/peak table — Degreaser consumes results downstream of peak identification
  and spectral matching
- The goal is QC on spectral purity, %RSD across replicates, or dilution-series linear
  range rather than retention-time plausibility; those are separate Degreaser evidence
  types
- A lipid class has fewer than the minimum 5 examples, so no ECN vs. RT regression
  can be fit for it
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipiDex 2
  role: Source of the Degreaser QC module whose ECN vs. RT modeling is being reproduced
    and timed; algorithms implemented in C# .NET 6
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Degreaser QC module
  role: Defines the ECN vs. RT regression (RT ~ FA carbons + alkene unsaturations),
    the 5-example-per-class minimum, and the 6 underlying QC data models being reproduced
  repo: https://github.com/coongroup/LipiDex-2
- name: C# .NET 6
  role: Runtime in which the LipiDex 2 QC calculations execute; timing benchmark compares
    against this implementation's speed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:28a9b977aeb81bf6df86b2b3539f6379b1c58ccae953816a77e461856d0b7ccb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C# .NET 6
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# qc-workflow-timing-reproduction

## Summary

Reproduces and benchmarks the Degreaser ECN vs. RT retention-time quality-control model from LipiDex 2, fitting per-lipid-class regressions of RT on fatty acyl carbon count and alkene unsaturations and recording the compute time for the full QC model build.

## When to use

Use when you need to reconstruct or benchmark a lipid retention-time QC model from a peak/feature table (e.g. LipiDex 2 demo dataset) and verify that model fitting completes within a target runtime such as 10 seconds.

## When NOT to use

- Input is a raw Thermo .raw or .mzML file rather than a post-identification lipid feature/peak table — Degreaser consumes results downstream of peak identification and spectral matching
- The goal is QC on spectral purity, %RSD across replicates, or dilution-series linear range rather than retention-time plausibility; those are separate Degreaser evidence types
- A lipid class has fewer than the minimum 5 examples, so no ECN vs. RT regression can be fit for it

## Inputs

- LipiDex 2 lipid feature results table (.csv peak table with lipid identifications, retention times, and fatty acyl compositions)
- LipiDex 2 demo dataset (Nile rat plasma data, e.g. MSV000091033-derived results)

## Outputs

- RT_model_report.csv with per-class regression coefficients, number of examples used, fit statistics, and elapsed calculation time
- Per-identification flags for large deviation from predicted retention time

## How to apply

Load the lipid feature results (peak table with identifications, retention times, and fatty acyl compositions) as produced by the LipiDex 2 pipeline's peak identification and spectral matching steps. Parse each lipid identification to extract fatty acyl carbon count and alkene unsaturation count, then group features by lipid class. Exclude classes with fewer than the Degreaser default of 5 examples, and fit the ECN vs. RT regression (RT ~ FA carbons + alkene unsaturations) per class, mirroring Degreaser's effective carbon number versus retention time modeling. Compute per-class coefficients and flag identifications whose retention time deviates largely from the predicted RT, since the model is used to filter likely incorrect identifications. Time the full model-fitting calculation across all 6 QC visualizations (the article reports ~3000 lipid features completes in under 10 seconds on a basic computer) and write class, coefficients, examples used, fit statistics, and timing to RT_model_report.csv.

## Related tools

- **LipiDex 2** (Source of the Degreaser QC module whose ECN vs. RT modeling is being reproduced and timed; algorithms implemented in C# .NET 6) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Degreaser QC module** (Defines the ECN vs. RT regression (RT ~ FA carbons + alkene unsaturations), the 5-example-per-class minimum, and the 6 underlying QC data models being reproduced) — https://github.com/coongroup/LipiDex-2
- **C# .NET 6** (Runtime in which the LipiDex 2 QC calculations execute; timing benchmark compares against this implementation's speed)

## Examples

```
python fit_rt_models.py --input lipid_features.csv --min-examples 5 --formula 'RT ~ FA_carbons + FA_alkenes' --group-by lipid_class --report RT_model_report.csv
```

## Evaluation signals

- RT_model_report.csv is produced with class-level coefficients, example counts, fit statistics, and a recorded calculation time
- Recorded model-fitting time on ~3000 lipid features is under 10 seconds, consistent with the reported LipiDex 2 performance
- Lipid classes with fewer than 5 examples are excluded from regression fitting (Degreaser default minimum)
- Per-class model direction is sensible: predicted RT increases with fatty acyl carbon count and decreases with alkene unsaturation count
- Identifications with large deviation from predicted RT are flagged, matching Degreaser's RT-filter behavior

## Limitations

- Classes with fewer than 5 examples (user-configurable default) cannot be modeled and are silently excluded
- The <10 s timing is implementation-dependent (C# .NET 6 on a basic computer); re-implementations in other languages may differ in speed
- The RT model only filters identifications with large deviation from predicted RT; it does not validate spectra, %RSD, or linear dynamic range

## Evidence
