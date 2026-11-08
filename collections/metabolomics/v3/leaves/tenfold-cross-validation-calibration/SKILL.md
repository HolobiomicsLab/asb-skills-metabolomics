---
name: tenfold-cross-validation-calibration
description: Use when user-supplied reference standard retention times (e.g. from
  MassBank records) are available and a per-dataset RT–log P model must be calibrated
  and validated before scoring MetFrag candidates by retention.
when_to_use_negative:
- No reference standards with measured retention times are available for the dataset,
  so the RT–log P model (Eq. 9) cannot be established
- Analysis is on non-reverse-phase LC where log P and retention time cannot be correlated
  due to column properties
- Candidates already come with a single pre-computed scoring scheme and no retention
  information is to be incorporated
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetFrag2.2
  role: Consumes the calibrated RT–log P score as a user-defined score in candidate
    ranking; retention scores are written to RetentionScore.csv
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: CDK XlogP
  role: Descriptor used to compute the octanol–water partition coefficient (log P)
    for reference standards and candidates feeding the RT–log P regression
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:353968ec662915d2c23e93461ffac4b77222f8fd2b8fe55c34cb4aecd4403b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - CDK XlogP
  - MetFrag2.2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tenfold-cross-validation-calibration

## Summary

Establishes and validates a dataset-specific linear RT–log P retention model (Eq. 9) by regressing reference-standard retention times on computed log P values (e.g. CDK XlogP) using tenfold cross-validation, so the model can be inverted to produce per-candidate retention scores for MetFrag2.2 ranking.

## When to use

Use when user-supplied reference standard retention times (e.g. from MassBank records) are available and a per-dataset RT–log P model must be calibrated and validated before scoring MetFrag candidates by retention.

## When NOT to use

- No reference standards with measured retention times are available for the dataset, so the RT–log P model (Eq. 9) cannot be established
- Analysis is on non-reverse-phase LC where log P and retention time cannot be correlated due to column properties
- Candidates already come with a single pre-computed scoring scheme and no retention information is to be incorporated

## Inputs

- Reference standard retention times per substance (e.g. extracted from MassBank records)
- Computed log P values for the standards (CDK XlogP descriptor or PubChem XLOGP3)
- Observed retention time of the unknown compound
- Retrieved candidate structures with per-candidate log P values

## Outputs

- Dataset-specific fitted linear RT–log P model coefficients (Eq. 9)
- Per-candidate retention scores written to RetentionScore.csv for use as a MetFrag user-defined score
- Validated score weight combination (ωFrag+ωRT+ωRefs=1) from 1000 randomly drawn combinations

## How to apply

Collect the retention times of reference standards for the dataset (in the MetFrag2.2 evaluation these were extracted from MassBank records per substance) and compute each standard's octanol–water partition coefficient (log P) with the CDK XlogP descriptor (or PubChem XLOGP3). Fit the linear model log P = a·RT + b (Eq. 9) by regressing the observed retention times on the log P values, and establish the RT–log P model for each dataset independently based on a tenfold cross-validation rather than pooling datasets. Invert the fitted model at the unknown's observed retention time to obtain the RT-implied log P, then score each candidate as the normal density (sigma = 1.5) of the deviation between the candidate's log P and the RT-implied log P. Write the per-candidate scores to RetentionScore.csv for use as a user-defined score in MetFrag candidate ranking. If combining scores, assess weight combinations by randomly drawing 1000 combinations such that (ωFrag+ωRT+ωRefs=1).

## Related tools

- **MetFrag2.2** (Consumes the calibrated RT–log P score as a user-defined score in candidate ranking; retention scores are written to RetentionScore.csv) — https://github.com/ipb-halle/MetFragRelaunched
- **CDK XlogP** (Descriptor used to compute the octanol–water partition coefficient (log P) for reference standards and candidates feeding the RT–log P regression)

## Examples

```
model <- lm(logP ~ RT, data = standards)  # Eq. 9: regress observed RTs on CDK XlogP values, then invert at the unknown's RT
```

## Evaluation signals

- The RT–log P model (Eq. 9) is established for each dataset independently and assessed by tenfold cross-validation, not pooled across datasets
- Candidate retention scores equal the normal density with sigma = 1.5 evaluated at the deviation between candidate log P and the RT-implied log P
- A RetentionScore.csv file exists with one score per candidate and is passed to MetFrag as a user-defined score
- Score weights satisfy the constraint ωFrag+ωRT+ωRefs=1 over 1000 randomly drawn combinations
- Model inversion at the unknown's observed RT yields a log P value consistent with the fitted regression range

## Limitations

- The RT–log P model must be established per dataset independently; it does not transfer across datasets or LC setups
- Retention times rely on the availability of reference standards (e.g. MassBank records); too few standards destabilize the regression
- Weight optimization by 1000 random draws is a heuristic search over combinations summing to 1 and does not guarantee the global optimum

## Evidence

- [results] The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [intro] the octanol–water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties: "the octanol–water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties"
- [results] the combination of each scoring term was assessed by randomly drawing 1000 different weight combinations such that (ωFrag+ωRT+ωRefs=1): [evidence span withheld — anchor status: not_found]
- [abstract] or incorporated separately as "user-defined scores" to be included in candidate ranking: "or incorporated separately as "user-defined scores" to be included in candidate ranking"
