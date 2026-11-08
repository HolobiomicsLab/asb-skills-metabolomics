---
name: normal-density-scoring-candidates
description: Use when a MetFrag2.2 run has user-supplied reference standard retention
  times plus measured RT of the unknown, and candidates should be ranked using the
  log P vs RT correlation term (Eq. 9-10).
when_to_use_negative:
- No user-supplied reference standard retention times are available, so the RT-log
  P regression of Eq. 9 cannot be fitted
- Separation is not on a reverse-phase LC column, where the article's premise that
  'the octanol-water partitioning coefficient (log P) and retention times (RT) of
  substances can be correlated due to the column properties' does not hold
- The goal is scoring by spectral fragmentation only, when retention information should
  not be included as a user-defined score in candidate ranking
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3520
tools:
- name: MetFrag2.2
  role: Consumes RetentionScore.csv as a user-defined score and combines it with the
    fragmentation score and reference count in candidate ranking
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFrag
  role: Base in silico fragmenter whose retrieved candidates receive the RT-derived
    normal-density scores
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:46e391b80454edf44bf3862392e7a2607abbf33f6754621f53941148921b7ca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - MetFrag
  - MetFrag2.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Normal-density scoring of candidates

## Summary

Scores candidate structures by how well their octanol-water partition coefficient (log P) matches the log P implied by the unknown's measured retention time, using a linear RT-log P regression (Eq. 9) and a normal density function with sigma = 1.5 (Eq. 10). This implements the retention-time scoring term (omega_RT) of MetFrag2.2's candidate ranking on reverse-phase LC/MS data.

## When to use

Use when a MetFrag2.2 run has user-supplied reference standard retention times plus measured RT of the unknown, and candidates should be ranked using the log P vs RT correlation term (Eq. 9-10).

## When NOT to use

- No user-supplied reference standard retention times are available, so the RT-log P regression of Eq. 9 cannot be fitted
- Separation is not on a reverse-phase LC column, where the article's premise that 'the octanol-water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties' does not hold
- The goal is scoring by spectral fragmentation only, when retention information should not be included as a user-defined score in candidate ranking

## Inputs

- List of reference standard retention times with known identities (for CDK XlogP computation and Eq. 9 regression)
- Observed retention time of the unknown compound from reverse-phase LC/MS
- Candidate structure list (SDF or database IDs from PubChem/ChemSpider/KEGG) for XlogP calculation
- MetFrag parameter file with the user-defined score (RetentionScore.csv) entry

## Outputs

- RetentionScore.csv containing the normal-density RT score per candidate
- Fitted RT-log P linear model parameters (a, b) from Eq. 9
- RT-implied log P estimate for the unknown compound

## How to apply

First compute each reference standard's octanol-water partition coefficient with the CDK XlogP descriptor (or PubChem XLOGP3 as an alternative), then fit the linear model log P = a*RT + b by least squares regression on the user-supplied RTs (Eq. 9). Invert the fitted model at the unknown's observed retention time to obtain the RT-implied log P. For each retrieved candidate structure, calculate its CDK XlogP value and score it as the normal density function with sigma = 1.5 evaluated at the deviation between the candidate's log P and the RT-implied log P (Eq. 10). Write the per-candidate scores to RetentionScore.csv and supply the file to MetFrag as a user-defined score so it is combined with the fragmentation score (omega_Frag + omega_RT + omega_Refs = 1). In the article's benchmark the RT-log P model was established per dataset independently and validated by tenfold cross-validation, and weight combinations were assessed by randomly drawing 1000 weight combinations summing to 1.

## Related tools

- **MetFrag2.2** (Consumes RetentionScore.csv as a user-defined score and combines it with the fragmentation score and reference count in candidate ranking) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFrag** (Base in silico fragmenter whose retrieved candidates receive the RT-derived normal-density scores) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
library(devtools); install_github("ipb-halle/MetFragR/metfRag")  # then compute CDK XlogP, fit log P = a*RT + b, score candidates with dnorm(dev, 0, 1.5), write RetentionScore.csv
```

## Evaluation signals

- RetentionScore.csv is produced with exactly one numeric RT score per candidate structure
- All scores lie in (0, 1/(1.5*sqrt(2*pi))] since they are normal density values with sigma = 1.5 at non-negative log P deviations
- The fitted Eq. 9 regression has plausible slope sign (higher log P implies longer RT on a reverse-phase column) and its tenfold cross-validation error is acceptable
- When the RT score is included in ranking (with weights summing to omega_Frag + omega_RT + omega_Refs = 1), the correct candidate's rank improves, as seen on the Hill et al. dataset where top-1 correct candidates rose from 8 to 9 and top-3 from 20 to 24

## Limitations

- The RT-log P correlation relies on reverse-phase column properties; it degrades when the regression is fitted from few reference standards or from MassBank-record RTs measured under different conditions
- CDK XlogP may be unavailable or unreliable for some candidate structures, forcing reliance on alternative estimators such as PubChem XLOGP3
- Retention scoring is only one term: the article's weight study (1000 random weight combinations with omega_Frag + omega_RT + omega_Refs = 1) shows results depend on the chosen weights, and mass spectra dominated by few fragments remain hard to resolve even with retention information

## Evidence

- [intro] Retention time information via log P correlation is used for candidate selection in LC/MS on reverse phase columns.: "the octanol–water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties"
- [results] The RT-log P model (Eq. 9) was established per dataset and validated by tenfold cross-validation.: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [abstract] Retention-time scores are supplied to MetFrag as user-defined scores in candidate ranking.: "or incorporated separately as "user-defined scores" to be included in candidate ranking"
- [results] Weight optimization randomly drew 1000 combinations with omega_Frag + omega_RT + omega_Refs = 1.: [evidence span withheld — anchor status: not_found]
- [readme] MetFragR package can be installed from GitHub for scripted MetFrag runs.: "install_github("ipb-halle/MetFragR/metfRag")"
