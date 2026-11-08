---
name: preprocessing-parameter-recording
description: Use when a notame preprocessing pipeline has produced a feature-by-sample
  abundance matrix and you must capture the exact transformation, PQN and center/scale
  settings before writing the transformed matrix onward.
when_to_use_negative:
- Input is raw instrument data (e.g. mzML or MS-DIAL peak tables) that has not yet
  been through notame's peak-picking, drift correction, flagging and imputation —
  parameter recording is only meaningful for a matrix that already has preprocessing
  parameters to describe.
- The analysis is purely exploratory visualization on an already-normalized matrix
  where no new transformation parameters were chosen — no parameter record needs to
  be written.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides pqn_normalization and the MetaboSet object whose exprs matrix is
    transformed, normalized and autoscaled; the recorded parameters describe calls
    to this package.
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Execution environment in which the notame preprocessing and parameter recording
    are performed.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:c0b8afae6ce5b26cd533fb3346f72d47839af418a8fef697071e7864338d75aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# preprocessing-parameter-recording

## Summary

Records the transformation, normalization and scaling parameters applied to a MetaboSet LC-MS feature matrix (e.g. log transformation, PQN reference/method, autoscaling center/scale flags) so that downstream multivariate analyses remain reproducible and auditable.

## When to use

Use when a notame preprocessing pipeline has produced a feature-by-sample abundance matrix and you must capture the exact transformation, PQN and center/scale settings before writing the transformed matrix onward.

## When NOT to use

- Input is raw instrument data (e.g. mzML or MS-DIAL peak tables) that has not yet been through notame's peak-picking, drift correction, flagging and imputation — parameter recording is only meaningful for a matrix that already has preprocessing parameters to describe.
- The analysis is purely exploratory visualization on an already-normalized matrix where no new transformation parameters were chosen — no parameter record needs to be written.

## Inputs

- MetaboSet object or exprs() feature-by-sample abundance matrix from full_data.RData
- chosen transformation type (natural log or glog)
- PQN reference sample set and normalization method
- center/scale flags for autoscaling

## Outputs

- transformed_exprs.csv — feature-by-sample numeric matrix after transformation, PQN and autoscaling
- transformation_parameters.json — parameter record with transformation type, PQN reference/method, and center/scale settings

## How to apply

After loading the preprocessed MetaboSet abundances from full_data.RData and extracting the exprs matrix with exprs(), apply the log transformation, then run pqn_normalization(object, ref, method, all_features), then autoscale by mean-centering and dividing by per-feature standard deviation. At each step write the parameter choices — transformation type (natural log or glog), the PQN reference and method, and the center/scale flags — into a structured record (transformation_parameters.json) rather than leaving them implicit in scripts. The record should name the function and the arguments actually passed (e.g. which samples form the PQN reference), mirroring the notame convention of keeping preprocessing parameters explicit so results can be traced back. Finally write the resulting matrix to transformed_exprs.csv alongside the parameter file so the pair is self-describing.

## Related tools

- **notame** (Provides pqn_normalization and the MetaboSet object whose exprs matrix is transformed, normalized and autoscaled; the recorded parameters describe calls to this package.) — https://github.com/hanhineva-lab/notame
- **R** (Execution environment in which the notame preprocessing and parameter recording are performed.)

## Examples

```
Rscript -e 'load("full_data.RData"); e <- exprs(object); e.log <- log(e); e.pqn <- pqn_normalization(e.log, ref, method="pqn"); e.scaled <- scale(e.pqn); write.csv(e.scaled, "transformed_exprs.csv"); jsonlite::write_json(list(transformation="log", pqn_ref=ref, pqn_method="pqn", center=TRUE, scale=TRUE), "transformation_parameters.json")'
```

## Evaluation signals

- transformed_exprs.csv exists and is a feature-by-sample numeric matrix with the same dimensions as the input exprs matrix.
- transformation_parameters.json exists and records the transformation type, the PQN reference and method, and the center/scale flags.
- The recorded PQN arguments correspond to an actual call of pqn_normalization(object, ref, method, all_features) in the pipeline.
- Expert review confirms the recorded parameter choices follow the Section 3.2.4 conventions of the notame paper.

## Limitations

- The package API is still quite experimental and breaking changes are possible, so recorded parameter names may need updating across notame versions.
- The paper notes preprocessing should be run on log-transformed data because it follows the model assumptions better and avoids practical issues; recording parameters of an untransformed matrix would miss this rationale.
- The parameter record does not itself validate the statistics — it only documents which transformation, PQN and scaling choices were made.

## Evidence

- [methods] Preprocessing on log-transformed data follows model assumptions and avoids practical issues: [evidence span withheld — anchor status: not_found]
- [readme] API is experimental, breaking changes possible: "Currently, the package is developed by Afekta Technologies. The package API is still quite experimental, and breaking changes are possible"
- [intro] bundle preprocessing methods into one package: "we use notame as a way to bundle together all the preprocessing methods we use for our non-targeted LC-MS metabolomics data"
- [methods] exprs part of the MetaboSet object: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
