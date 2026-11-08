---
name: tautomer-candidate-ranking
description: Use when a CASMI-style MS/MS challenge has the correct structure present
  in the PubChem candidate list but several fragment peaks stay unexplained under
  the 10 ppm/0.002 Da tolerance and tautomers exist in the candidate set.
when_to_use_negative:
- The unexplained peaks are due to the correct structure being absent from the candidate
  list (e.g. a halogenated or unusual molecule outside the C,H,N,O,P,S element filter)
  — relax the candidate retrieval or in silico fragmentation depth instead of only
  the mass tolerance.
- The peak list contains only stable, high-abundance fragments already matched under
  the default 10 ppm/0.002 Da tolerance with no tautomer ambiguity in the candidate
  set.
- The goal is molecular formula prediction (category 1 CASMI challenge), where the
  structure-level tautomer ranking is not the deliverable.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MAGMa
  role: Runs the candidate scoring and substructure matching; invoked in command line
    mode with an overridden minimum mass tolerance of 0.005 Da to re-rank tautomers
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Supplies the candidate molecule set retrieved from a local PubChem snapshot
    (2 December 2013) whose tautomers are being re-ranked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:31aee5dda5e61f90d666b478e76aafc920e5792b361e940c24b10261f036589c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tautomer-candidate-ranking

## Summary

Ranks chemically equivalent tautomer candidates by MAGMa penalty score rather than accepting a failed annotation as a missing candidate: when MS/MS peaks remain unexplained under the default mass tolerance (10 ppm, absolute minimum 0.002 Da), re-run candidate scoring with a relaxed minimum mass tolerance (0.005 Da) so alternative tautomers of the correct structure surface at the top of the ranking.

## When to use

Use when a CASMI-style MS/MS challenge has the correct structure present in the PubChem candidate list but several fragment peaks stay unexplained under the 10 ppm/0.002 Da tolerance and tautomers exist in the candidate set.

## When NOT to use

- The unexplained peaks are due to the correct structure being absent from the candidate list (e.g. a halogenated or unusual molecule outside the C,H,N,O,P,S element filter) — relax the candidate retrieval or in silico fragmentation depth instead of only the mass tolerance.
- The peak list contains only stable, high-abundance fragments already matched under the default 10 ppm/0.002 Da tolerance with no tautomer ambiguity in the candidate set.
- The goal is molecular formula prediction (category 1 CASMI challenge), where the structure-level tautomer ranking is not the deliverable.

## Inputs

- MAGMa input file with precursor and MS/MS fragment peaks in hierarchical m/z:intensity parenthesis format
- Local PubChem candidate database snapshot (2 December 2013) filtered to elements C,H,N,O,P,S and mass ≤ 1200 Da

## Outputs

- Ranked candidate list per challenge (e.g. magma_challenge11_0.005da_ranking.csv) with penalty score and rank per candidate
- Identification of which tautomer(s) occupy top positions after relaxing the mass tolerance

## How to apply

1. Load the challenge's composite MS/MS input (precursor plus fragment peaks in MAGMa's hierarchical parenthesis format) over candidates from the local PubChem database (compounds on 2 December 2013; elements C,H,N,O,P,S; ≤1200 Da). 2. First run with the default allowed mass deviation of 10 ppm, absolute minimum 0.002 Da, substructures from up to 3 bond dissociations plus single losses of hydroxyl or primary amine groups, and confirm whether the correct tautomer candidates are actually present but poorly ranked. 3. If unexplained peaks persist, raise the minimum mass tolerance to 0.005 Da and re-process, so that peaks like m/z 119.0538, 143.0534 and 158.0396 find acceptable substructure matches. 4. Recompute the intensity-weighted average total penalty score per candidate (including penalty 10 for non-matched peaks) and rank candidates by reversed penalty score, ordering equal scores by PubChem refscore. 5. Confirm that the tautomer pair occupies the top ranked positions (e.g. rows '<500 candidates' rank 1 and '<200 candidates' rank 2 in Table 1) before declaring the annotation solved.

## Related tools

- **MAGMa** (Runs the candidate scoring and substructure matching; invoked in command line mode with an overridden minimum mass tolerance of 0.005 Da to re-rank tautomers) — https://github.com/NLeSC/MAGMa
- **PubChem** (Supplies the candidate molecule set retrieved from a local PubChem snapshot (2 December 2013) whose tautomers are being re-ranked)

## Examples

```
magma --input challenge11.msms --mass-tolerance 0.005 --output magma_challenge11_0.005da_ranking.csv
```

## Evaluation signals

- The two demethoxycurcumin tautomers occupy ranks 1 and 2 in the re-processed challenge 11 candidate list.
- Peaks previously left unexplained (m/z 119.0538, 143.0534, 158.0396) now have acceptable substructure matches under the 0.005 Da minimum tolerance.
- No candidate outside the tautomer pair outperforms them on the intensity-weighted average total penalty score.
- Equal-scoring candidates are ordered by PubChem refscore, not arbitrarily.

## Limitations

- Raising the minimum mass tolerance above the default 10 ppm/0.002 Da risks admitting lower-quality substructure matches; the article frames the 0.002 Da default as too restrictive rather than recommending a universal relaxation.
- This skill presupposes the correct structure is present in the candidate set; if it is missing (as in challenge 16, where default in silico fragmentation was insufficient), relaxing tolerance alone will not solve the annotation.
- Penalty score 10 is assigned to each non-matched peak, so candidates with many unexplained peaks can be heavily penalised when tolerance remains too strict.

## Evidence

- [results] correct tautomers present but missed by initial tolerance: [evidence span withheld — anchor status: not_found]
- [methods] penalty calculation: [evidence span withheld — anchor status: not_found]
- [methods] ranking rule: [evidence span withheld — anchor status: not_found]
