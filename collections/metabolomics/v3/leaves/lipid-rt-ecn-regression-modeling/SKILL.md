---
name: lipid-rt-ecn-regression-modeling
description: Use when a lipid feature table with identifications, retention times,
  and fatty acyl compositions needs per-class RT plausibility checks or filtering
  of likely incorrect identifications.
when_to_use_negative:
- Input is a raw MS file or unannotated peak table with no lipid identifications and
  fatty acyl compositions — the regression needs parsed FA carbon counts and unsaturations
  per class
- The goal is generic chromatographic peak detection or alignment rather than lipid-identification
  plausibility QC
- A lipid class has fewer than the minimum examples (default 5), making a per-class
  regression unreliable
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2 Degreaser QC module
  role: Implements effective carbon number versus retention time (ECN vs. RT) modeling
    as one of 6 automated QC checks and filters identifications deviating from predicted
    RT
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2
  role: Pipeline whose peak identification and spectral matching results serve as
    input to the ECN vs. RT model; rewritten in C# .NET 6
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:39e76dd9faa794448d7ba121a21a5b5e9785636e4a58f4e6a063694be7a3588c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C# .NET 6
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# lipid-rt-ecn-regression-modeling

## Summary

Per-lipid-class regression of retention time against fatty acyl composition (RT ~ FA carbons + alkene unsaturations, i.e., effective carbon number vs. RT modeling) as implemented in the LipiDex 2 Degreaser QC module to flag identifications with large deviation from predicted RT.

## When to use

Use when a lipid feature table with identifications, retention times, and fatty acyl compositions needs per-class RT plausibility checks or filtering of likely incorrect identifications.

## When NOT to use

- Input is a raw MS file or unannotated peak table with no lipid identifications and fatty acyl compositions — the regression needs parsed FA carbon counts and unsaturations per class
- The goal is generic chromatographic peak detection or alignment rather than lipid-identification plausibility QC
- A lipid class has fewer than the minimum examples (default 5), making a per-class regression unreliable

## Inputs

- LipiDex 2 pipeline peak table with lipid identifications, retention times, and fatty acyl compositions (Degreaser takes results from the peak identification and spectral matching steps)
- User-specified minimum number of examples per lipid class (default 5)

## Outputs

- Per-lipid-class ECN vs. RT regression coefficients and predicted retention times
- Flags on lipid identifications with large deviation from predicted RT (RT model filter)
- RT model report (class, coefficients, examples used, fit statistics, timing), e.g., RT_model_report.csv

## How to apply

Load the lipid feature results (peak table with identifications, retention times, and fatty acyl compositions), parse each identification to extract fatty acyl carbon counts and alkene unsaturations, and group features by lipid class. Exclude classes with fewer than the user-set minimum number of examples (default 5 in Degreaser), then fit an ECN-based linear regression (RT ~ FA carbons + alkene unsaturations) independently within each lipid class. Compute per-class coefficients and flag identifications whose measured RT deviates largely from the predicted RT; these can be filtered as likely incorrect identifications. Degreaser performs this as one of 6 pieces of QC evidence, and the underlying data models for all 6 visualizations compute in under 10 seconds for ~3000 lipid features on a basic computer.

## Related tools

- **LipiDex 2 Degreaser QC module** (Implements effective carbon number versus retention time (ECN vs. RT) modeling as one of 6 automated QC checks and filters identifications deviating from predicted RT) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2** (Pipeline whose peak identification and spectral matching results serve as input to the ECN vs. RT model; rewritten in C# .NET 6) — https://github.com/coongroup/LipiDex-2

## Examples

```
LipiDex2.exe  # run the Degreaser QC module on the peak identification results; ECN vs. RT modeling is one of its 6 automated checks
```

## Evaluation signals

- A per-class ECN vs. RT model report is produced listing class-level coefficients, number of examples used, and fit statistics
- Lipid classes with fewer than 5 examples (default minimum) are excluded from model fitting
- Identifications with large deviation from predicted RT are flagged/filtered as likely incorrect
- Model-fitting across ~3000 lipid features completes in under 10 seconds on a basic computer
- Downstream filtered results show removal of RT-outlier identifications among the 7 filtering criteria

## Limitations

- Classes with fewer than the minimum examples (default 5) are excluded from modeling
- RT plausibility is a proxy — a plausible RT does not guarantee a correct identification; it only flags large deviations
- LipiDex 2 wiki documentation was a work in progress ('WiP') at the time of the README

## Evidence

- [methods] Degreaser models retention times within a lipid class with RT ~ FA alkene unsaturations + FA carbons, applying a user-specific minimum number of examples per class (default 5): [evidence span withheld — anchor status: not_found]
