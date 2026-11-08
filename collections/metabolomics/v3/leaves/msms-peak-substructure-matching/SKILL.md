---
name: msms-peak-substructure-matching
description: Use when annotating unknown fragment ion spectra solely from candidate
  structures in the absence of reference spectra, e.g. ranking PubChem candidates
  for LC-MS metabolite identification challenges.
when_to_use_negative:
- When high-quality reference spectra exist for a database match (MAGMa's value is
  structure-only annotation without reference spectra)
- When the elemental composition includes elements outside C/H/N/O/P/S, since the
  local PubChem snapshot used for candidate retrieval was restricted to those elements
- When the spectrum contains peaks requiring more than 3 bond dissociations (e.g.
  challenge 16 higher-energy peaks up to m/z=221.07309 remained unexplained)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MAGMa
  role: 'Core engine: fragments candidates in silico (up to 3 bond dissociations),
    matches peaks to substructures, and computes penalty scores for ranking'
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecule structures; candidates retrieved from a local
    snapshot of all PubChem compounds on 2 December 2013, filtered by precursor mass
    tolerance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:6a29c5d73235913110a68a8742264b5a204b81b2f7f1197bc76f0bcfe65007be
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# msms-peak-substructure-matching

## Summary

Score candidate molecular structures by matching each MS/MS peak m/z value against in silico generated substructures (up to 3 bond dissociations, plus neutral water/ammonia losses), yielding intensity-weighted penalty scores used to rank candidate molecules.

## When to use

Use when annotating unknown fragment ion spectra solely from candidate structures in the absence of reference spectra, e.g. ranking PubChem candidates for LC-MS metabolite identification challenges.

## When NOT to use

- When high-quality reference spectra exist for a database match (MAGMa's value is structure-only annotation without reference spectra)
- When the elemental composition includes elements outside C/H/N/O/P/S, since the local PubChem snapshot used for candidate retrieval was restricted to those elements
- When the spectrum contains peaks requiring more than 3 bond dissociations (e.g. challenge 16 higher-energy peaks up to m/z=221.07309 remained unexplained)

## Inputs

- MS/MS (or multistage MSn) peak lists with m/z and intensity values, in MAGMa hierarchical input format
- Candidate molecule structures (SDF) retrieved from a PubChem snapshot restricted to C/H/N/O/P/S
- Precursor m/z with allowed mass deviation (e.g. 10 ppm, minimum 0.002 Da)

## Outputs

- Penalty score per candidate molecule (intensity-weighted average of substructure penalties)
- Ranked list of candidate molecules (reversed penalty order, refscore tie-breaker)
- Top-ranked molecular formula and/or chemical structure for each challenge

## How to apply

Convert the CASMI-style peak lists to MAGMa hierarchical input (multistage fragmentation trees with fragment peaks nested in parentheses), then retrieve candidate molecules from the local PubChem snapshot (2 December 2013, C/H/N/O/P/S only, mass <= 1200 Da) using a mass deviation of 10 ppm with an absolute minimum of 0.002 Da (4 ppm for challenges 4-6). For each candidate, generate substructures by applying up to 3 bond dissociations and expand the set with single losses of hydroxyl groups (neutral water loss) or primary amine groups (neutral ammonia loss). Match each observed peak m/z against the best-scoring substructure, compute per-peak penalties (10 for non-matched peaks), and calculate a total penalty score as an intensity-weighted average over all peaks. Rank candidates in reversed penalty-score order, breaking ties by PubChem refscore; the top-ranked candidate's formula is the annotation output.

## Related tools

- **MAGMa** (Core engine: fragments candidates in silico (up to 3 bond dissociations), matches peaks to substructures, and computes penalty scores for ranking) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecule structures; candidates retrieved from a local snapshot of all PubChem compounds on 2 December 2013, filtered by precursor mass tolerance)

## Examples

```
magma job --input challenge_01.magma --candidates pubchem_2013-12-02 --mass-tolerance 10ppm
```

## Evaluation signals

- All top-ranked molecular formulas submitted in category 1 (8 challenges) were correct when comparing against CASMI ground truth
- In 6 of 12 submitted category 2 solutions the correct candidate ranked first; 3 more fell within the top 5 — ranking quality can be sanity-checked against these baselines
- Verify that the total penalty score equals the intensity-weighted average of per-peak substructure penalties, with non-matched peaks assigned penalty 10
- Confirm candidate retrieval used the correct mass tolerance (10 ppm / 0.002 Da minimum; 4 ppm for challenges 4-6) and that substructures included water and ammonia neutral losses

## Limitations

- Too-restrictive mass tolerance (0.002 Da) caused unexplained fragment peaks in challenge 11 even though correct tautomers were present in the candidate list
- Default in-silico fragmentation depth (3 bond dissociations) was insufficient for challenge 16, leaving several higher-energy peaks (m/z up to 221.07309) unexplained
- Successful application depends on the relevant candidate molecule being present in the initial candidate set; refscore thresholding (e.g. threshold 5) may remove true candidates when candidate lists are large

## Evidence

- [methods] neutral loss expansion: [evidence span withheld — anchor status: not_found]
- [methods] penalty aggregation: [evidence span withheld — anchor status: not_found]
- [methods] ranking: [evidence span withheld — anchor status: not_found]
- [results] challenge 16 limitation: [evidence span withheld — anchor status: not_found]
- [results] challenge 11 limitation: [evidence span withheld — anchor status: not_found]
