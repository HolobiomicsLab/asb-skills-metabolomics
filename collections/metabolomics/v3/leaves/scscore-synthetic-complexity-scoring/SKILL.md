---
name: scscore-synthetic-complexity-scoring
description: Use when filtering enumerated derivative functional groups (e.g., substituents
  applied n times to a drug template) so that only synthetically accessible groups
  survive into the NPS enumeration, using ΔSCScore thresholds (α^n, α = 1.2).
when_to_use_negative:
- The goal is predicting mass spectra (NEIMS) or molecular fingerprints (DeepEI) of
  already-filtered derivatives — this skill only scores synthetic accessibility, not
  spectral properties.
- Enumerated compounds are already constrained by the other rule-based filters (e.g.,
  Messy rings with more than 7 smallest sets of smallest rings, the 57 drug-like criteria
  of Virshup et al.) and no complexity-based screening is required.
- The analyte identification stage where unknown GC/EI-MS spectra are matched against
  the synthetic database — this skill belongs to the database-construction phase only.
edam_topics:
- http://edamontology.org/topic_3343
tools:
- name: SCScore
  role: Pretrained neural-network model used to compute the synthetic complexity score
    of the cathinone template baseline and of each functional-group-applied derivative
    compound
- name: rdkit
  role: Chemistry toolkit used to handle SMILES structures and apply functional groups
    to the cathinone template during iterative derivative generation
  repo: https://www.rdkit.org/docs/Install.html
- name: python 3.6.9 or higher
  role: Runtime for executing the SCScore deep learning tools in the PS2MS pipeline
  repo: https://www.python.org/downloads/
- name: PS2MS
  role: Pipeline whose enumeration stage supplies the 204 functional groups and cathinone-templated
    derivatives that this filtering step scores and screens
  repo: https://github.com/jhhung/PS2MS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:8bc0ad1666bea3979b7e6a2802837ad8997f2a10d5276f1804c5d3e38b4aec3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PS2MS
  - SCScore
  - python
  - python 3.6.9 or higher
  - rdkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3343
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scscore-synthetic-complexity-scoring

## Summary

Computes SCScore-based synthetic-difficulty differentials (ΔSCScore) for enumerated cathinone-derivative functional groups and retains only those whose average score falls below a recursion-dependent threshold, preventing highly challenging-to-synthesize compounds from inflating the synthetic NPS database.

## When to use

Use when filtering enumerated derivative functional groups (e.g., substituents applied n times to a drug template) so that only synthetically accessible groups survive into the NPS enumeration, using ΔSCScore thresholds (α^n, α = 1.2).

## When NOT to use

- The goal is predicting mass spectra (NEIMS) or molecular fingerprints (DeepEI) of already-filtered derivatives — this skill only scores synthetic accessibility, not spectral properties.
- Enumerated compounds are already constrained by the other rule-based filters (e.g., Messy rings with more than 7 smallest sets of smallest rings, the 57 drug-like criteria of Virshup et al.) and no complexity-based screening is required.
- The analyte identification stage where unknown GC/EI-MS spectra are matched against the synthetic database — this skill belongs to the database-construction phase only.

## Inputs

- Enumerated functional-group list (204 compiled functional groups) and cathinone-templated derivative set from the enumeration step of PS2MS
- Cathinone core structure (SMILES, processed with RDKit)
- Pretrained SCScore neural-network model

## Outputs

- Per-functional-group score table with ΔSCScore values and per-level pass/fail flags written to scscore_filter_results.csv
- 131 functional groups passing the threshold at recursion level n = 2 and 97 passing at n = 3

## How to apply

Load the enumerated functional-group list and the cathinone-templated derivative set (204 compiled functional groups from the enumeration stage of PS2MS). Compute the SCScore of the cathinone template as the baseline synthetic-difficulty reference using the pretrained SCScore neural-network model. For each functional group, apply it iteratively n times to the template (n = 2 and n = 3), compute the SCScore of each resulting compound, and average the scores per functional group to obtain its synthesis-difficulty differential ΔSCScore relative to the baseline. Retain a functional group only if ΔSCScore < α^n with α = 1.2 applied per recursion level, and record per-level pass/fail flags. The rationale is that functional groups whose repeated application drives synthesis complexity far above the template would include highly challenging-to-synthesize compounds in the enumeration process and waste downstream NEIMS/DeepEI prediction effort.

## Related tools

- **SCScore** (Pretrained neural-network model used to compute the synthetic complexity score of the cathinone template baseline and of each functional-group-applied derivative compound)
- **rdkit** (Chemistry toolkit used to handle SMILES structures and apply functional groups to the cathinone template during iterative derivative generation) — https://www.rdkit.org/docs/Install.html
- **python 3.6.9 or higher** (Runtime for executing the SCScore deep learning tools in the PS2MS pipeline) — https://www.python.org/downloads/
- **PS2MS** (Pipeline whose enumeration stage supplies the 204 functional groups and cathinone-templated derivatives that this filtering step scores and screens) — https://github.com/jhhung/PS2MS

## Evaluation signals

- The number of functional groups passing the ΔSCScore < α^n retention rule equals 131 at the second recursion level (n = 2).
- The number of functional groups passing at the third recursion level (n = 3) equals 97.
- scscore_filter_results.csv contains one row per functional group with a ΔSCScore value and explicit pass flags for both levels n = 2 and n = 3.
- The baseline SCScore is computed on the cathinone template itself, and every per-group ΔSCScore is a differential relative to that baseline.
- All retained functional groups satisfy ΔSCScore < 1.2^n for their respective recursion level; spot-check that no retained group exceeds the threshold.

## Limitations

- The threshold parameters (α = 1.2, evaluated at n = 2 and n = 3) are tied to the cathinone template; applying the same rule to other drug templates may not reproduce comparable pass counts.
- SCScore provides a relative synthetic-difficulty proxy, not a guarantee of synthetic feasibility; compounds with low ΔSCScore may still be difficult to synthesize in practice.
- The paper does not report specific accession IDs or release versions for the synthetic NPS database or the pretrained SCScore model used.

## Evidence

- [methods] We then calculated the average synthetic complexity score of the resulting compounds using SCScore.: "We then calculated the average synthetic complexity score of the resulting compounds using SCScore."
- [methods] retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [discussion] We utilize these scores to screen functional groups, thus avoiding the inclusion of highly challenging-to-synthesize compounds in the enumeration process: "We utilize these scores to screen functional groups, thus avoiding the inclusion of highly challenging-to-synthesize compounds in the enumeration process"
