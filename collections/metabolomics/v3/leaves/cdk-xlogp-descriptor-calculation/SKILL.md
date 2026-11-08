---
name: cdk-xlogp-descriptor-calculation
description: Use when ranking LC-MS/MS candidate structures where reference standards
  with measured RTs are available on a reverse-phase column and retention-time plausibility
  should contribute to MetFrag scoring.
when_to_use_negative:
- Analysis is not LC-based on a reverse-phase column, so RT and log P are not meaningfully
  correlated
- No reference standards with measured retention times are available to fit the RT–log
  P model
- Candidates are gas-phase or GC-MS identifications where retention indices, not log
  P, drive retention behaviour
tools:
- name: CDK XlogP
  role: Descriptor used to compute octanol–water partition coefficients (log P) for
    reference standards and candidate structures in the RT–log P regression
- name: PubChem XLOGP3
  role: Alternative log P source mentioned alongside CDK XlogP for the retention-time
    score
- name: MetFrag2.2
  role: Consumes the computed XlogP-derived retention-time scores as a user-defined
    score (RetentionScore.csv) in candidate ranking
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragR (metfRag)
  role: R interface to MetFrag that can be used to run MetFrag scoring from R where
    the log P regression is computed
  repo: http://c-ruttkies.github.io/MetFrag/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:71c25b6cd99c1af262ef0618284d29324bd8aecace67e9ba9184e51d212dcb90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CDK XlogP
  - MetFrag2.2
  - MetFragR (metfRag)
  - PubChem XLOGP3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cdk-xlogp-descriptor-calculation

## Summary

Calculates octanol–water partition coefficients (log P) with the CDK XlogP descriptor (or PubChem XLOGP3 as an alternative) to link retention times to candidate structures in MetFrag2.2. The log P values feed the RT–log P regression used for retention-time scoring of candidate metabolites.

## When to use

Use when ranking LC-MS/MS candidate structures where reference standards with measured RTs are available on a reverse-phase column and retention-time plausibility should contribute to MetFrag scoring.

## When NOT to use

- Analysis is not LC-based on a reverse-phase column, so RT and log P are not meaningfully correlated
- No reference standards with measured retention times are available to fit the RT–log P model
- Candidates are gas-phase or GC-MS identifications where retention indices, not log P, drive retention behaviour

## Inputs

- List of reference standard retention times (user-supplied, from reverse-phase LC runs or MassBank records)
- Chemical structures of reference standards (SMILES/SDF, for XlogP computation)
- Retrieved candidate structures (PubChem/ChemSpider hits or user SDF)
- Observed retention time of the unknown compound

## Outputs

- Fitted linear RT–log P regression parameters (a, b)
- RT-implied log P for the unknown compound
- Per-candidate CDK XlogP values
- RetentionScore.csv with one retention-time score per candidate (user-defined score for MetFrag ranking)

## How to apply

For each user-supplied reference standard retention time, compute the octanol–water partition coefficient using the CDK XlogP descriptor (PubChem XLOGP3 is an acceptable alternative source). Fit the linear model log P = a·RT + b (Eq. 9) by regressing the standards' retention times on their XlogP values, establishing the model independently per dataset — in the MetFrag2.2 evaluation this was done via tenfold cross-validation using RTs from MassBank records. Invert the fitted model at the unknown's observed RT to obtain the RT-implied log P, then compute the XlogP of each retrieved candidate structure. Score each candidate as the normal density function with sigma = 1.5 evaluated at the deviation between the candidate's log P and the RT-implied log P, and export the per-candidate values to RetentionScore.csv for use as a user-defined score in the MetFrag candidate ranking.

## Related tools

- **CDK XlogP** (Descriptor used to compute octanol–water partition coefficients (log P) for reference standards and candidate structures in the RT–log P regression)
- **PubChem XLOGP3** (Alternative log P source mentioned alongside CDK XlogP for the retention-time score)
- **MetFrag2.2** (Consumes the computed XlogP-derived retention-time scores as a user-defined score (RetentionScore.csv) in candidate ranking) — http://c-ruttkies.github.io/MetFrag/
- **MetFragR (metfRag)** (R interface to MetFrag that can be used to run MetFrag scoring from R where the log P regression is computed) — http://c-ruttkies.github.io/MetFrag/

## Examples

```
model <- lm(logP ~ RT, data = standards); logP_unknown <- coef(model)[1] + coef(model)[2] * RT_unknown; score <- dnorm(cand_logP - logP_unknown, sd = 1.5); write.csv(score, "RetentionScore.csv")
```

## Evaluation signals

- The RT–log P linear fit holds across the standards, and tenfold cross-validation of the model per dataset does not reveal systematic misfit
- RetentionScore.csv exists with one normal-density score per candidate (sigma = 1.5) and no missing entries for candidates with computable XlogP
- Adding the retention score to MetFrag ranking improves candidate ranks, as in the Hill et al. dataset where top-1/3/5 correct candidates improved from 8/20/28 to 9/24/34
- XlogP values for standards computed via CDK and via PubChem XLOGP3 agree sufficiently that the fitted model is not descriptor-source dependent

## Limitations

- The RT–log P model must be established independently per dataset/column; a model fitted on one LC setup does not transfer
- The model is only valid for reverse-phase LC where retention correlates with the octanol–water partitioning coefficient due to column properties
- CDK XlogP and PubChem XLOGP3 can differ; the article names both as possible log P sources, so descriptor choice must be consistent between standards and candidates

## Evidence

- [intro] RT–log P correlation rationale: "the octanol–water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties"
- [results] RT–log P model established per dataset: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [full_text] log P sources for the retention score: [evidence span withheld — anchor status: not_found]
- [readme] metfRag R interface installation: "install_github("ipb-halle/MetFragR/metfRag")"
