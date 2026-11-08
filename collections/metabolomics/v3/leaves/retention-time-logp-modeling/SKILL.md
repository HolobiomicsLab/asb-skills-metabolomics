---
name: retention-time-logp-modeling
description: Use when LC/MS HR-MS/MS candidates must be re-ranked using measured retention
  times on a reverse-phase column and reference-standard RTs are available to fit
  the log P correlation.
when_to_use_negative:
- No reference-standard retention times exist for the compound class, so the RT–log
  P regression (Eq. 9) cannot be fit
- Data come from direct infusion or non-reverse-phase LC where log P and retention
  time are not correlated due to column properties
- Only HR-MS/MS peak matching is needed without retention-time information (then plain
  MetFrag2.2/CFM-ID fragmenter scoring suffices)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetFrag2.2
  role: Fits the RT–log P model, computes candidate XlogP, writes RetentionScore.csv
    and consumes it as a user-defined score in candidate ranking
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFragCommandLine
  role: Command-line MetFrag2.2 variant built with Maven to run the RT-scored ranking
    on the 473-spectrum Eawag Orbitrap XL dataset
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: PubChem
  role: Source of candidate structures (downloaded June 2015) and reference/data-source/patent
    information combined with the retention score
- name: ChemSpider
  role: Alternative candidate database providing reference and data-source information
    alongside the retention-time score
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:77e36b49f10046bf484a3308ff943d270519ab5730863756671c146bbd2bb5d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - CFM-ID
  - ChemSpider
  - MetFrag2.2
  - MetFragCommandLine
  - PubChem
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# retention-time-logp-modeling

## Summary

Builds a linear RT–log P retention model (Eq. 9) from reference-standard retention times and octanol–water partition coefficients, then converts a candidate's CDK XlogP deviation from the RT-implied log P into a normal-density score used as a user-defined score in MetFrag2.2 candidate ranking.

## When to use

Use when LC/MS HR-MS/MS candidates must be re-ranked using measured retention times on a reverse-phase column and reference-standard RTs are available to fit the log P correlation.

## When NOT to use

- No reference-standard retention times exist for the compound class, so the RT–log P regression (Eq. 9) cannot be fit
- Data come from direct infusion or non-reverse-phase LC where log P and retention time are not correlated due to column properties
- Only HR-MS/MS peak matching is needed without retention-time information (then plain MetFrag2.2/CFM-ID fragmenter scoring suffices)

## Inputs

- Reference-standard retention times (e.g. from MassBank records) for substances in the dataset
- Observed retention time of the unknown compound from the Orbitrap XL LC-MS/MS run
- Candidate structures retrieved from ChemSpider/PubChem web services or user-supplied SDF
- MetFrag2.2 settings file configuring the user-defined retention score

## Outputs

- Fitted RT–log P regression parameters (slope a, intercept b) per dataset
- RetentionScore.csv with a per-candidate normal-density (sigma = 1.5) retention score
- Score-ranked candidate list with Top 1 rank counts after stereoisomer filtering

## How to apply

Collect the reference-standard retention times per substance (e.g. extracted from MassBank records) and compute each standard's octanol–water partition coefficient with the CDK XlogP descriptor; fit the linear model log P = a·RT + b (Eq. 9) by least squares, establishing the model independently per dataset with tenfold cross-validation. Invert the fitted model at the unknown's observed retention time to obtain the RT-implied log P. For each candidate structure retrieved from ChemSpider or PubChem (June 2015 snapshot in the benchmark), calculate its CDK XlogP and score it as the normal density function with sigma = 1.5 evaluated at the deviation between candidate log P and the RT-implied log P. Write the per-candidate values to RetentionScore.csv and supply them to MetFrag as a user-defined score, combined with the fragmenter, reference and data-source/patent scores under the constraint (ωFrag+ωRT+ωRefs=1) optimized by randomly drawing 1000 weight combinations. After ranking, filter stereoisomers by keeping only the best-scored candidate per first InChIKey block before counting Top 1 ranks.

## Related tools

- **MetFrag2.2** (Fits the RT–log P model, computes candidate XlogP, writes RetentionScore.csv and consumes it as a user-defined score in candidate ranking) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFragCommandLine** (Command-line MetFrag2.2 variant built with Maven to run the RT-scored ranking on the 473-spectrum Eawag Orbitrap XL dataset) — https://github.com/ipb-halle/MetFragRelaunched
- **PubChem** (Source of candidate structures (downloaded June 2015) and reference/data-source/patent information combined with the retention score)
- **ChemSpider** (Alternative candidate database providing reference and data-source information alongside the retention-time score)

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git && mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- RetentionScore.csv exists with one finite normal-density score (sigma = 1.5) per candidate structure
- Fitted regression parameters are established per dataset via tenfold cross-validation, not pooled across datasets
- Combined weights satisfy ωFrag+ωRT+ωRefs=1 after drawing 1000 random combinations
- On the 473-spectrum Eawag Orbitrap XL dataset with reference/RT scoring, Top 1 ranks reach ~420/473 (89%) with ChemSpider and ~336/473 (71%) with PubChem, ~343 with CFM-ID added
- Ranked lists show stereoisomers collapsed to one representative per first InChIKey block

## Limitations

- The RT–log P model must be re-established independently per dataset; scores are not transferable across columns or chromatographic methods
- Requires sufficiently many reference standards with measured RTs; identifications lacking a pure reference standard cannot be confirmed
- Score-based re-ranking cannot rescue mass spectra dominated by one or a few fragments that most candidates explain

## Evidence

- [results] RT–logP model fitting: "The retention times extracted from the MassBank records per substance were used to establish the RT–log P model (see Eq. 9) for each dataset independently based on a tenfold cross-validation"
- [intro] rationale for RT–logP correlation: "the octanol–water partitioning coefficient (log P) and retention times (RT) of substances can be correlated due to the column properties"
- [results] weight optimization constraint: [evidence span withheld — anchor status: not_found]
- [readme] MetFragCommandLine dependency for running the workflow: "MetFrag commandline version depends on MetFragLib"
- [results] stereoisomer filtering after scoring: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
