---
name: ppm-mass-error-preannotation
description: Use when you have a list of theoretical metabolite m/z values from in-silico
  reaction-rule expansion and a feature table of dysregulated peaks, and need to flag
  which features correspond to which reaction-derived candidates.
when_to_use_negative:
- Input is already a validated, standard-confirmed annotation set — 5 ppm preannotation
  is only a first-pass cross-reference, not confirmation
- Only MS2 spectra are available without accurate MS1 m/z values, since the matching
  operates entirely on monoisotopic precursor masses
- The goal is discovery of intact (non-biotransformed) parent chemicals against a
  reference library, where traditional bottom-up library search is appropriate
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: CMDN
  role: Framework whose reaction-based pseudo-MS1 candidature space (RCS) layer performs
    the reaction-rule expansion and 5 ppm m/z preannotation
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: biotransformation prediction
  role: In-silico generation of biotransformed derivatives whose computed monoisotopic
    m/z values feed the candidature space
- name: LC-HRMS
  role: Source of the dysregulated feature m/z values against which theoretical candidates
    are matched
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:781f084ab1cb5bcc2090bf2bf3b87347593262d7c1ba8d17e93a2ad392d18240
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - biotransformation prediction
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# ppm-mass-error-preannotation

## Summary

Preannotates theoretical biotransformed metabolite candidates against dysregulated LC-HRMS features by accurate-mass matching within a tight m/z error tolerance (≤5 ppm), as implemented in the RCS layer of CMDN.

## When to use

Use when you have a list of theoretical metabolite m/z values from in-silico reaction-rule expansion and a feature table of dysregulated peaks, and need to flag which features correspond to which reaction-derived candidates.

## When NOT to use

- Input is already a validated, standard-confirmed annotation set — 5 ppm preannotation is only a first-pass cross-reference, not confirmation
- Only MS2 spectra are available without accurate MS1 m/z values, since the matching operates entirely on monoisotopic precursor masses
- The goal is discovery of intact (non-biotransformed) parent chemicals against a reference library, where traditional bottom-up library search is appropriate

## Inputs

- Parent-compound structure library (e.g., 1,021-pesticide structure library)
- Curated nonisomeric phase-I reaction rules (290 KEGG/literature-derived rules)
- Dysregulated feature table from the differential metabolite space (DMS) layer of LC-HRMS data

## Outputs

- rcs_candidate_features.csv with per-feature theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm
- Nonredundant set of theoretical biotransformed derivatives (pseudo-MS1 candidature space)

## How to apply

Load a parent-compound structure library (e.g., the 1,021-pesticide library used in CMDN) and a curated set of nonisomeric phase-I reaction rules (290 rules adopted in CMDN, sourced from KEGG and curated literature). Apply each rule in silico to every parent structure to enumerate theoretical biotransformed derivatives and compute monoisotopic m/z values, forming the pseudo-MS1 candidature space (RCS). Load the dysregulated features flagged by the differential metabolite space (DMS) layer, then cross-reference theoretical candidate m/z values against those features using a mass-error tolerance of ≤5 ppm. Emit one row per preannotated feature containing theoretical m/z, parent identity, applied reaction rule, and the m/z error in ppm. Correctness hinges on every emitted entry satisfying the 5 ppm criterion and the nonredundant candidate set being consistent in scale with the expected annotation outcome (2,886 nonredundant derivatives of 1,021 pesticides).

## Related tools

- **CMDN** (Framework whose reaction-based pseudo-MS1 candidature space (RCS) layer performs the reaction-rule expansion and 5 ppm m/z preannotation) — https://github.com/LeaveMeNotTonight/CMDN
- **biotransformation prediction** (In-silico generation of biotransformed derivatives whose computed monoisotopic m/z values feed the candidature space)
- **LC-HRMS** (Source of the dysregulated feature m/z values against which theoretical candidates are matched)

## Evaluation signals

- Every cross-referenced entry in rcs_candidate_features.csv has |m/z error| ≤ 5 ppm
- The enumerated nonredundant candidate set is consistent in scale with the reported 2,886 nonredundant biotransformed derivatives of 1,021 pesticides
- Each preannotated feature carries the applied reaction rule and parent pesticide identity (schema completeness check)
- Theoretical candidate set contains no duplicate derivatives (nonredundancy invariant)

## Limitations

- 5 ppm mass matching is preannotation only; candidates require downstream validation (standard verification, retention time prediction, murine studies, time-course logistic modeling) per CMDN's multichannel validation
- Reaction-rule expansion is bounded by the 290 nonisomeric phase-I rules adopted in CMDN; metabolites outside these rules are missed
- Theoretical enumeration can be large (2,886 nonredundant derivatives), so false positives may pass the 5 ppm filter and require MS2 similarity network (SSN) disambiguation

## Evidence

- [methods] Theoretical candidate features generated by the RCS reaction rules were cross-referenced with dysregulated features identified in the DMS layer and preannotated by accurate mass matching at an m/z error tolerance of ≤ 5 ppm.: [evidence span withheld — anchor status: not_found]
