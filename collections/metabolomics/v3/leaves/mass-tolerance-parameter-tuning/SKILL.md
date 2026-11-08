---
name: mass-tolerance-parameter-tuning
description: Use when candidate-structure annotation leaves high-quality MS/MS peaks
  unexplained and you must decide whether the failure stems from a missing candidate
  structure or from too-restrictive mass tolerance settings.
when_to_use_negative:
- The correct candidate was verified absent from the initial PubChem candidate set
  — raising the mass tolerance cannot recover a missing structure
- Peaks remain unexplained even with relaxed tolerance because the default in silico
  fragmentation depth (up to 3 bond dissociations) is insufficient, as in challenge
  16 where higher-energy peaks like m/z 122.0408 and 221.07309 stayed unmatched
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MAGMa
  role: Substructure-based annotation engine run in command line mode; its mass deviation
    parameter (default 10 ppm, minimum 0.002 Da) is overridden to 0.005 Da for the
    tolerance test
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules retrieved from the local database snapshot of
    2 December 2013; refscore used to order equal-scoring candidates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:033c30fda2f72ddd1bb19400fef91eb1c778772ce1a035d6549a07d3eb6d0ee8
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
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-tolerance-parameter-tuning

## Summary

A diagnostic-and-replay procedure for MAGMa candidate ranking: when MS/MS peaks remain unexplained, test whether the assumed mass deviation (default 10 ppm with an absolute minimum of 0.002 Da) was too restrictive rather than assuming the correct structure was absent from the candidate list, by reprocessing with a raised minimum mass tolerance (0.005 Da).

## When to use

Use when candidate-structure annotation leaves high-quality MS/MS peaks unexplained and you must decide whether the failure stems from a missing candidate structure or from too-restrictive mass tolerance settings.

## When NOT to use

- The correct candidate was verified absent from the initial PubChem candidate set — raising the mass tolerance cannot recover a missing structure
- Peaks remain unexplained even with relaxed tolerance because the default in silico fragmentation depth (up to 3 bond dissociations) is insufficient, as in challenge 16 where higher-energy peaks like m/z 122.0408 and 221.07309 stayed unmatched

## Inputs

- MAGMa input file for the challenge (precursor peak plus MS/MS fragment peaks in hierarchical m/z:intensity parenthesis format)
- Candidate molecule set from a local PubChem database (snapshot 2 December 2013; elements C,H,N,O,P,S; up to 1200 Da)
- Original MAGMa run parameters (mass deviation 10 ppm, absolute minimum 0.002 Da; up to 3 bond dissociations; hydroxyl/primary amine neutral losses)

## Outputs

- Reprocessed ranked candidate list (e.g. magma_challenge11_0.005da_ranking.csv) with intensity-weighted penalty scores
- Matching assignments for previously unexplained MS/MS peaks to theoretical substructure m/z values
- A conclusion on root cause: tolerance too restrictive vs. correct structure absent from candidate list

## How to apply

Load the challenge's MAGMa input file (precursor plus fragment peaks in the hierarchical m/z:intensity parenthesis format) and run MAGMa in command line mode over candidates retrieved from the local PubChem database (compounds on 2 December 2013, elements C,H,N,O,P,S, up to 1200 Da), keeping the default substructure generation of up to 3 bond dissociations plus single losses of hydroxyl or primary amine groups. Override the allowed mass deviation — default 10 ppm with an absolute minimum of 0.002 Da — to a fixed minimum mass tolerance of 0.005 Da so that previously unexplained fragment peaks can match theoretical substructure m/z values. Match each MS/MS peak with the best scoring substructures, compute the intensity-weighted average total penalty score per candidate (penalty 10 for non-matched peaks), rank by reversed penalty score, and order equal scores by PubChem refscore. Compare the new ranking against the original: if the correct structure moves into the top ranks and the unexplained peaks (e.g. m/z 119.0538, 143.0534, 158.0396 for challenge 11) now match, the original tolerance was too restrictive. Save the reprocessed ranked candidate list for auditability.

## Related tools

- **MAGMa** (Substructure-based annotation engine run in command line mode; its mass deviation parameter (default 10 ppm, minimum 0.002 Da) is overridden to 0.005 Da for the tolerance test) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules retrieved from the local database snapshot of 2 December 2013; refscore used to order equal-scoring candidates)

## Examples

```
magma --input challenge11.ms --candidates pubchem_20131202 --mass-tolerance 0.005 --output magma_challenge11_0.005da_ranking.csv
```

## Evaluation signals

- Previously unexplained peaks (challenge 11: m/z 119.0538, 143.0534, 158.0396) gain acceptable substructure matches at 0.005 Da minimum tolerance
- The correct structure (challenge 11: demethoxycurcumin tautomers) ranks 1st and 2nd in the reprocessed ranking, matching the '<500 candidates' rank 1 and '<200 candidates' rank 2 pattern in Table 1
- The ranked CSV preserves the scoring scheme: intensity-weighted average penalty with penalty 10 for non-matched peaks, ties broken by PubChem refscore
- All other run parameters (3 bond dissociations, hydroxyl/amine neutral losses, candidate database snapshot) are unchanged from the original run, isolating the tolerance variable

## Limitations

- Raising the tolerance can only fix failures caused by overly restrictive mass deviation (10 ppm / 0.002 Da); it cannot compensate for a correct structure missing from the candidate list
- Some peaks remain unexplained even after tolerance relaxation when the fragmentation depth is insufficient (challenge 16 higher-energy peaks required more than 3 bond dissociations)
- MAGMa's allowed mass deviation is defined as 10 ppm with an absolute minimum of 0.002 Da, so low-m/z fragments below ~20 Da effectively need the absolute-Da override rather than a ppm setting

## Evidence

- [results] Retrospective diagnosis of challenge 11: [evidence span withheld — anchor status: not_found]
- [results] Fragmentation depth is a separate failure mode: [evidence span withheld — anchor status: not_found]
- [methods] Command line mode of operation: [evidence span withheld — anchor status: not_found]
