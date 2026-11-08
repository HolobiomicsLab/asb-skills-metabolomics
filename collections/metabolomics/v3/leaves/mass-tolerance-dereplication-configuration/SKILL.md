---
name: mass-tolerance-dereplication-configuration
description: Reference for setting the mzdiff mass tolerance in XCMS CentWave when
  features with similar m/z and retention times need dereplication, or when dereplication
  should be disabled.
when_to_use_negative:
- When the analysis does not use XCMS CentWave feature extraction, since mzdiff is
  specific to that program's dereplication step.
- When the goal is tuning peak height or other universal parameters (e.g., peak width,
  instrumental shift) rather than mass-tolerance dereplication.
- When a different dereplication strategy (not based on m/z/retention-time similarity
  within a mass tolerance) is required.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: XCMS CentWave
  role: Feature-extraction program whose dereplication step is controlled by the mzdiff
    mass tolerance
- name: Paramounter
  role: R program that measures universal parameters and translates them into software-specific
    settings, including mzdiff
  repo: https://github.com/HuanLab/Paramounter
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c04758
    title: 'Paramounter: Direct Measurement of Universal Parameters To Process Metabolomics
      Data in a “White Box”'
  source_content_hash: sha256:2abc9dc8287c6a4327fd8d6ed9d4db659fe97530dd6a5b2c5da56a1eb4d11944
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/Paramounter
  tools:
  - Paramounter
  - R
  - XCMS CentWave
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-tolerance-dereplication-configuration

## Summary

Configure the XCMS CentWave mzdiff mass-tolerance parameter that dereplicates features with similar m/z and retention times, choosing 0.001–0.01 as suggested defaults or a negative value to disable dereplication entirely.

## When to use

Reference for setting the mzdiff mass tolerance in XCMS CentWave when features with similar m/z and retention times need dereplication, or when dereplication should be disabled.

## When NOT to use

- When the analysis does not use XCMS CentWave feature extraction, since mzdiff is specific to that program's dereplication step.
- When the goal is tuning peak height or other universal parameters (e.g., peak width, instrumental shift) rather than mass-tolerance dereplication.
- When a different dereplication strategy (not based on m/z/retention-time similarity within a mass tolerance) is required.

## Inputs

- Feature list extracted by XCMS CentWave (m/z values and retention times)
- Paramounter-measured universal parameters including mass tolerance
- Intended dereplication behavior (enabled vs. disabled)

## Outputs

- Configured mzdiff value (0.001, 0.01, or a negative value to disable dereplication)
- XCMS CentWave parameter set reflecting the chosen mass tolerance

## How to apply

In XCMS CentWave, mzdiff acts as the mass tolerance that dereplicates extracted features (similar m/z values and retention times). Use a suggested default of 0.001 or 0.01 to enable dereplication, accepting that true positive metabolic features with mass differences smaller than the threshold may be removed by mistake. If the dataset contains many near-identical features and any true positives would be lost, set mzdiff to any negative value to disable the dereplication function. When tuning, balance the false-removal risk against redundant feature reporting; this choice interacts with Paramounter's optimized peak height threshold, which can be raised (e.g., 2X the optimized value) to reduce false positives independently of mzdiff.

## Related tools

- **XCMS CentWave** (Feature-extraction program whose dereplication step is controlled by the mzdiff mass tolerance)
- **Paramounter** (R program that measures universal parameters and translates them into software-specific settings, including mzdiff) — https://github.com/HuanLab/Paramounter

## Examples

```
Rscript paramounter_tune.R --mzdiff 0.001
```

## Evaluation signals

- Verify the chosen mzdiff value is 0.001, 0.01, or negative; no other positive value is suggested by the documentation.
- If dereplication is enabled, confirm that near-identical features (similar m/z and retention time within mzdiff) are merged in the output feature table.
- If dereplication is disabled (negative mzdiff), confirm that features with small mass differences are retained rather than removed.
- Cross-check that the mzdiff setting does not silently increase false positives when combined with Paramounter's optimized peak height threshold.
- Confirm the configured parameter set is accepted by XCMS CentWave without errors when the pipeline runs.

## Limitations

- Suggested default mzdiff values (0.001 or 0.01) can remove true positive metabolic features whose mass differences are smaller than the tolerance.
- Disabling dereplication (negative mzdiff) may leave redundant near-identical features in the output.
- Paramounter's optimized peak height threshold can raise false positive rates and the likelihood of software crash; a higher threshold (e.g., 2X) is recommended to mitigate this, independent of mzdiff.

## Evidence
