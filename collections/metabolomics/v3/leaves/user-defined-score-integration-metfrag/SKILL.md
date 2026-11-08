---
name: user-defined-score-integration-metfrag
description: Use when ranking MetFrag candidates for an LC-MS unknown with a measured
  retention time on a reverse-phase column and a set of reference standards whose
  RTs and structures (for log P) are available.
when_to_use_negative:
- No measured retention time for the unknown or too few reference standards with known
  RTs and structures to fit the RT–log P regression
- Chromatography is not reverse-phase LC, so the correlation between log P and retention
  time 'due to the column properties' does not hold
- The candidate list is already ranked with an external RT score and you only need
  MetFrag fragmentation scoring without user-defined scores
tools:
- name: MetFrag2.2
  role: Fragments candidates in silico, ingests RetentionScore.csv as a user-defined
    score, and combines it with the fragmentation score in candidate ranking
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragR (metfRag)
  role: R interface to MetFragLib for running candidate retrieval/ranking from R so
    the externally computed RT scores can be supplied as user-defined scores
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:b75b6193b91ad4e8e37a9a4383a8b04447ba0f5073f58f512323255189314758
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFrag2.2
  - MetFragR (metfRag)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# User-defined score integration in MetFrag (retention-time score via log P regression)

## Summary

Computes a per-candidate retention-time score for MetFrag by fitting a linear RT–log P model from reference standards (Eqs. 9–10) and evaluating a normal density (sigma = 1.5) on the log P deviation between the unknown's RT-implied log P and each candidate's log P, then feeding the scores back into MetFrag as a user-defined score.

## When to use

Use when ranking MetFrag candidates for an LC-MS unknown with a measured retention time on a reverse-phase column and a set of reference standards whose RTs and structures (for log P) are available.

## When NOT to use

- No measured retention time for the unknown or too few reference standards with known RTs and structures to fit the RT–log P regression
- Chromatography is not reverse-phase LC, so the correlation between log P and retention time 'due to the column properties' does not hold
- The candidate list is already ranked with an external RT score and you only need MetFrag fragmentation scoring without user-defined scores

## Inputs

- List of reference standard retention times (LC-MS, reverse-phase column) and their structures for log P calculation (CDK XlogP or PubChem XLOGP3)
- Observed retention time of the unknown compound
- Candidate structure list retrieved from PubChem/ChemSpider/KEGG or a user-supplied SDF
- MetFrag parameter file specifying the user-defined score file and scoring weights (ωFrag, ωRT, ωRefs)

## Outputs

- RetentionScore.csv with the per-candidate RT score (normal density, sigma = 1.5, of the log P deviation)
- Ranked candidate list from MetFrag combining fragmentation score and user-defined RT score with weights summing to 1

## How to apply

Compute the octanol–water partition coefficient (log P) of each user-supplied reference standard (CDK XlogP or PubChem XLOGP3) and fit the linear model log P = a·RT + b by least-squares regression on their measured retention times (Eq. 9); the article establishes such RT–log P models per dataset and validates them by tenfold cross-validation. Invert the fitted model at the unknown's observed RT to get the RT-implied log P, and calculate each candidate's CDK XlogP. Score each candidate as the normal density with sigma = 1.5 evaluated at the deviation between candidate log P and the RT-implied log P, write these to RetentionScore.csv, and supply that file to MetFrag as a user-defined score. Combine scoring terms with weights (ωFrag + ωRT + ωRefs = 1); in the paper the weights were optimized by randomly drawing 1000 combinations summing to 1. On the Hill et al. dataset this RT information improved the median rank from 18.5 to 14.5 and top-1/3/5 correct candidates from 8/20/28 to 9/24/34 versus MetFrag2010.

## Related tools

- **MetFrag2.2** (Fragments candidates in silico, ingests RetentionScore.csv as a user-defined score, and combines it with the fragmentation score in candidate ranking) — http://c-ruttkies.github.io/MetFrag/
- **MetFragR (metfRag)** (R interface to MetFragLib for running candidate retrieval/ranking from R so the externally computed RT scores can be supplied as user-defined scores) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
m <- lm(logP ~ RT, data=stds); lp <- predict(m, data.frame(RT=RT_unk)); write.csv(dnorm(cand_logP - lp, 0, 1.5), "RetentionScore.csv")
```

## Evaluation signals

- RT–log P linear model is established per dataset and assessed by tenfold cross-validation, as done for the MassBank-derived retention times
- Every candidate in the final ranking carries a finite RT score in RetentionScore.csv (normal density value with sigma = 1.5)
- Scoring weights satisfy ωFrag + ωRT + ωRefs = 1 (paper explored 1000 random weight combinations)
- Ranking improves relative to fragmentation-only MetFrag2010 (e.g. Hill dataset: median rank 18.5 → 14.5, top-1/3/5 correct 8→9, 20→24, 28→34)
- Spectra are only ranked if they contain at least one fragment peak other than the precursor

## Limitations

- The RT–log P correlation is valid only for reverse-phase LC columns, as it depends on column properties
- Mass spectra dominated by one or a few fragments (e.g. a water loss) lack information to distinguish candidates, so the RT score alone may not resolve them
- Retention-time models were built per dataset (tenfold cross-validation) — a model fitted on one dataset/gradient should not be transferred naively to another

## Evidence

- [abstract] Incorporation of retention time information as user-defined scores in candidate ranking: "or incorporated separately as "user-defined scores" to be included in candidate ranking"
- [results] RT–logP model per dataset with tenfold cross-validation: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [results] Weight optimization for combined scoring terms: [evidence span withheld — anchor status: not_found]
- [intro] Rationale: log P and RT are correlated on reverse-phase columns: "the octanol–water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties"
- [readme] R package installation for programmatic MetFrag use: "install_github("ipb-halle/MetFragR/metfRag")"
