---
name: log-transform-robust-scaling-of-retention-times
description: Use when preparing RT pairs (e.g. DNN predictions from SMRT vs experimental
  PredRet RTs) for GP-based projection or regression fitting, so skewed RTs are transformed
  to a comparable scale.
when_to_use_negative:
- When RTs are already log-quantile transformed as part of a different pipeline (the
  RT-prediction step uses quantile transformation of RTs before fitting instead)
- When modeling only non-retained molecules (RT < 5 min), which are excluded from
  projection validation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: cmmrt
  role: Package implementing the RT preprocessing (log transform + robust scaling)
    and GP projection workflow
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Execution environment for the cmmrt preprocessing pipeline
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:fda842efed5adc7c00da71b44b8f411347039972d5ded651c89adca038b7d982
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - cmmrt
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# log-transform-robust-scaling-of-retention-times

## Summary

Preprocesses paired predicted/experimental retention times by log-transforming RTs and applying robust median/IQR scaling before fitting GP projection models, stabilizing the skewed RT distributions across chromatographic methods.

## When to use

Use when preparing RT pairs (e.g. DNN predictions from SMRT vs experimental PredRet RTs) for GP-based projection or regression fitting, so skewed RTs are transformed to a comparable scale.

## When NOT to use

- When RTs are already log-quantile transformed as part of a different pipeline (the RT-prediction step uses quantile transformation of RTs before fitting instead)
- When modeling only non-retained molecules (RT < 5 min), which are excluded from projection validation

## Inputs

- Paired predicted/experimental retention times (seconds) between a DNN predictor and a chromatographic method
- Predicted RT distribution of the fingerprint-trained DNN on SMRT (for learning the median/IQR scaling parameters)

## Outputs

- Log-transformed, robustly scaled RT pairs suitable for GP projection model fitting
- Fitted scaling parameters (median, IQR, 0.741 factor) reusable on held-out molecules

## How to apply

RTs are first transformed to log space via the log(1+x) transform, then further scaled using robust statistics (median and IQR). For projection validation, the median/IQR scaling (with 0.741 factor) is learned from the predicted RTs of the fingerprint-trained DNN on SMRT and then applied to both predicted and experimental RTs so both axes are comparable. The transformed pairs then serve as training points for the meta-learned GP prior over projection functions. Before transformation, molecules classified as non-retained (RT < 5 min) are excluded per the target-CM protocol.

## Related tools

- **cmmrt** (Package implementing the RT preprocessing (log transform + robust scaling) and GP projection workflow) — https://github.com/constantino-garcia/cmmrt
- **Python** (Execution environment for the cmmrt preprocessing pipeline) — https://github.com/constantino-garcia/cmmrt

## Examples

```
python cmmrt/rt/train_projections.py  # meta-train GP for projections; cmmrt applies log(1+x) transform and robust median/IQR scaling internally
```

## Evaluation signals

- Scaled RTs should have median approximately 0 and IQR-normalized spread ~1 on both predicted and experimental axes
- Projected RTs on held-out PredRet molecules should yield MAE comparable to the paper's reported per-CM errors in projection_results.csv
- Non-retained molecules (RT < 5 min) must be excluded before fitting the scaling parameters
- The same scaling parameters (median/IQR/0.741) must be applied identically to predicted and experimental RTs for the projection pairs

## Limitations

- Scaling parameters learned from predicted RTs assume the DNN prediction distribution is a reasonable proxy for the target CM's RT distribution
- Log(1+x) transform and robust scaling are tuned for reversed-phase LC-MS RTs; molecules not retained (RT < 5 min) are excluded and degrade the mapping if included

## Evidence

- [methods] Log-transformation of RTs: "RTs are transformed to log space using"
- [methods] Robust scaling with median and IQR: "data is further scaled using robust statistics."
- [other] Leave-one-CM-out validation workflow: [evidence span withheld — anchor status: not_found]
- [methods] Non-retained molecule exclusion: "a molecule was considered non-retained if its Retention Time (RT) was smaller than 5 minutes."
- [abstract] cmmrt package availability: "Python code is available on GitHub at https://github.com/constantino-garcia/cmmrt"
