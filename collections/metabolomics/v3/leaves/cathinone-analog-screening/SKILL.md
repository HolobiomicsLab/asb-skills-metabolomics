---
name: cathinone-analog-screening
description: Use when enumerating cathinone-templated derivative libraries and you
  must prune functional groups whose repeated substitution (n = 2, 3) makes derivatives
  too hard to synthesize, per ΔSCScore < α^n.
when_to_use_negative:
- The analyte mass spectrum already matches a reference library (NIST/Wiley/SWGDRUG),
  so conventional GC/EI-MS screening suffices and no synthetic NPS database needs
  to be built.
- Functional-group filtering is already complete and the task is downstream spectrum
  prediction (NEIMS) or fingerprint-based ranking (DeepEI) of enumerated derivatives.
- The drug template is not being recursively substituted (no n = 2 or n = 3 enumeration
  levels), so a per-level synthesis-difficulty differential cannot be computed.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: SCScore
  role: Pretrained neural network computing the synthetic complexity score of the
    cathinone template and each derivatized compound; averaged per functional group
    to yield ΔSCScore
- name: rdkit
  role: Molecular toolkit for applying functional groups to the cathinone template
    and handling derivative structures (installed from conda per the README)
  repo: https://www.rdkit.org/docs/Install.html
- name: PS2MS
  role: Pipeline in which the SCScore filter runs between the enumeration step and
    mass-spectrum generation of the synthetic NPS database
  repo: https://github.com/jhhung/PS2MS
- name: python 3.6.9
  role: Runtime for the SCScore deep-learning tools
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:2cf43fab85f2151f2ef9b31f4cb14af976b0a13530a8275cb1059e446c41879c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - python 3.6.9
  - rdkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cathinone-analog-screening

## Summary

Screen enumerated functional groups by synthesis difficulty using SCScore, retaining only groups whose average synthesis-difficulty differential (ΔSCScore) relative to the cathinone template falls below α^n (α = 1.2) at each enumeration recursion level. This prevents highly challenging-to-synthesize derivatives from inflating the synthetic NPS database.

## When to use

Use when enumerating cathinone-templated derivative libraries and you must prune functional groups whose repeated substitution (n = 2, 3) makes derivatives too hard to synthesize, per ΔSCScore < α^n.

## When NOT to use

- The analyte mass spectrum already matches a reference library (NIST/Wiley/SWGDRUG), so conventional GC/EI-MS screening suffices and no synthetic NPS database needs to be built.
- Functional-group filtering is already complete and the task is downstream spectrum prediction (NEIMS) or fingerprint-based ranking (DeepEI) of enumerated derivatives.
- The drug template is not being recursively substituted (no n = 2 or n = 3 enumeration levels), so a per-level synthesis-difficulty differential cannot be computed.

## Inputs

- Enumerated functional-group list (204 compiled functional groups) from the derivative-enumeration step
- Cathinone core-structure template (SMILES) for derivative generation and SCScore baseline
- Pretrained SCScore neural-network model

## Outputs

- scscore_filter_results.csv with per-functional-group ΔSCScore values and pass/fail flags at recursion levels 2 and 3
- Filtered functional-group set (131 groups at n=2, 97 groups at n=3) feeding downstream derivative enumeration

## How to apply

Load the enumerated functional-group list and cathinone-templated derivative set (204 compiled functional groups) produced by the enumeration step. Compute the SCScore of the cathinone template with the pretrained SCScore neural-network model as the baseline synthetic-difficulty reference. For each functional group, iteratively apply it n times to the template (n = 2 and n = 3), compute the SCScore of each resulting compound, and average per group to obtain ΔSCScore relative to the baseline. Retain the functional group only if ΔSCScore < α^n with α = 1.2 at each recursion level, flagging pass/fail at levels 2 and 3. Write the per-group score table with ΔSCScore values and per-level pass flags to scscore_filter_results.csv; validation expects 131 groups passing at level 2 and 97 at level 3.

## Related tools

- **SCScore** (Pretrained neural network computing the synthetic complexity score of the cathinone template and each derivatized compound; averaged per functional group to yield ΔSCScore)
- **rdkit** (Molecular toolkit for applying functional groups to the cathinone template and handling derivative structures (installed from conda per the README)) — https://www.rdkit.org/docs/Install.html
- **PS2MS** (Pipeline in which the SCScore filter runs between the enumeration step and mass-spectrum generation of the synthetic NPS database) — https://github.com/jhhung/PS2MS
- **python 3.6.9** (Runtime for the SCScore deep-learning tools) — https://www.python.org/downloads/

## Evaluation signals

- Number of functional groups passing the threshold equals 131 at the second recursion level (n=2) and 97 at the third (n=3), matching the reported counts.
- Every retained group satisfies ΔSCScore < α^n with α = 1.2 relative to the cathinone-template baseline SCScore.
- scscore_filter_results.csv contains a ΔSCScore row and per-level pass flags for all 204 input functional groups.
- Pass counts decrease monotonically from level 2 (131) to level 3 (97), consistent with stricter difficulty at higher substitution depth.
- No functional group with average synthesis-difficulty differential at or above α^n appears in the downstream enumeration set.

## Limitations

- The α = 1.2 threshold and recursion depths (n = 2, 3) are calibrated for the cathinone template; other templates may require retuning.
- SCScore is a proxy for synthetic complexity, so filtering excludes 'highly challenging-to-synthesize' compounds heuristically rather than guaranteeing actual synthesizability.
- The paper does not specify the exact number of enumeration levels beyond those tested, and the filtered database still contains ~422 million compounds, so downstream spectrum prediction remains large-scale.

## Evidence

- [methods] retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [methods] We then calculated the average synthetic complexity score of the resulting compounds using SCScore.: "We then calculated the average synthetic complexity score of the resulting compounds using SCScore."
- [discussion] We utilize these scores to screen functional groups, thus avoiding the inclusion of highly challenging-to-synthesize compounds in the enumeration process: "We utilize these scores to screen functional groups, thus avoiding the inclusion of highly challenging-to-synthesize compounds in the enumeration process"
- [readme] PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups.: "PS<sup>2</sup>MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
