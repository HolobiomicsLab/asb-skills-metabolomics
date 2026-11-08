---
name: in-silico-bond-dissociation-fragmentation
description: Use when MS/MS or MSn fragment spectra must be explained without reference
  spectra, e.g. CASMI-style compound annotation where candidate molecules need penalty-score
  ranking based on in silico generated substructures.
when_to_use_negative:
- Reference mass spectra are already available for the target metabolite, so peak-to-structure
  annotation can be done directly without in silico fragmentation scoring.
- The observed fragment m/z values clearly require neutral-loss ion series that MAGMa's
  bond-dissociation plus water/ammonia-loss expansion does not cover, and no extra
  fragmentation depth or substructure expansion is planned.
- Goal is de novo molecular formula generation rather than ranking a pre-retrieved
  candidate structure set.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: MAGMa
  role: Performs the systematic in silico fragmentation of candidate structures (up
    to 3 or 5 bond dissociations), matches generated substructures to MS/MS m/z values,
    and computes penalty-based candidate scores.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Supplies the candidate molecule structures (local database snapshot, e.g.
    2 December 2013) that MAGMa fragments; also provides the refscore used for tie-breaking
    candidate order.
- name: emetabolomics web interface
  role: Web front-end (www.emetabolomics.org/magma) through which MAGMa fragmentation
    jobs can be launched and results viewed as an alternative to command-line mode.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:07580b13ed0069ba0194ea26be1e3243647916e69e5e82e66031f03898f6db38
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# in-silico-bond-dissociation-fragmentation

## Summary

Systematically fragments candidate chemical structures in silico by applying successive single-bond dissociations (up to 3, or extended to 5 for retrospective deep fragmentation), producing substructure sets whose m/z values are matched against MS/MS peaks to score and rank candidates.

## When to use

Use when MS/MS or MSn fragment spectra must be explained without reference spectra, e.g. CASMI-style compound annotation where candidate molecules need penalty-score ranking based on in silico generated substructures.

## When NOT to use

- Reference mass spectra are already available for the target metabolite, so peak-to-structure annotation can be done directly without in silico fragmentation scoring.
- The observed fragment m/z values clearly require neutral-loss ion series that MAGMa's bond-dissociation plus water/ammonia-loss expansion does not cover, and no extra fragmentation depth or substructure expansion is planned.
- Goal is de novo molecular formula generation rather than ranking a pre-retrieved candidate structure set.

## Inputs

- MAGMa hierarchical input files (peak lists nested in parentheses after the precursor mono-isotopic peak) built from CASMI contest MS/MS spectra
- Candidate molecule structures retrieved from a local PubChem database in SDF format (element-filtered, e.g. C, H, N, O, P, S; halogenated supplement for challenges 15-16)
- Precursor molecular formula and measured accurate-mass MS/MS peak list at multiple collision energies

## Outputs

- Per-candidate substructure sets with assigned m/z and penalty scores
- Total penalty score (intensity-weighted average) per candidate molecule
- Ranked candidate list (reversed penalty score ordering, refscore tie-breaking), e.g. ranking tables such as retrospective_rankings.csv

## How to apply

Take each candidate molecule retrieved by mass search (e.g. 10 ppm deviation with a 0.002 Da absolute minimum, or 4 ppm for low-mass-precision challenges) and run MAGMa's systematic fragmentation, applying up to 3 bond dissociations by default and expanding the substructure set with single losses of hydroxyl (water) or primary amine (ammonia) groups. Match the m/z values of MS/MS or MSn peaks to the best-scoring substructures, where a substructure penalty is the sum of penalty values for all dissociated bonds. Compute a total penalty score per candidate as the intensity-weighted average of substructure penalties, assigning a penalty of 10 to non-matched peaks. Rank candidates by reversed penalty score, breaking ties by PubChem refscore. When high-energy peaks remain unexplained (as in CASMI challenge 16 with peaks up to m/z 221.07309), raise the dissociation depth to 5 bonds retrospectively and re-score; verify rankings reproduce the reported outcome.

## Related tools

- **MAGMa** (Performs the systematic in silico fragmentation of candidate structures (up to 3 or 5 bond dissociations), matches generated substructures to MS/MS m/z values, and computes penalty-based candidate scores.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Supplies the candidate molecule structures (local database snapshot, e.g. 2 December 2013) that MAGMa fragments; also provides the refscore used for tie-breaking candidate order.)
- **emetabolomics web interface** (Web front-end (www.emetabolomics.org/magma) through which MAGMa fragmentation jobs can be launched and results viewed as an alternative to command-line mode.)

## Examples

```
magma --candidates candidates.sdf --peaks peaks.magma --mass-deviation 10ppm --bond-dissociations 5
```

## Evaluation signals

- Correct candidate's ranking reproduces the expected penalty-score ordering: for CASMI challenge 16 re-run with 5-bond depth, ofloxacin must land at rank 126; challenge 13 aloxistatin drops from 42 to 58; challenge 15 2-(perfluorooctyl)ethanol moves to rank 1.
- Every matched m/z in the output has a corresponding best-scoring substructure whose penalty equals the sum of penalties of all dissociated bonds.
- Total penalty score per candidate equals the intensity-weighted average of substructure penalties, with a penalty of 10 assigned to each non-matched peak.
- Candidate set sizes match the reported filtering targets (e.g. <1000, <500, <200 molecules after refscore-threshold stepwise raising) when those thresholds are applied.
- Equally scoring candidates are ordered by PubChem refscore and tied candidates receive their average ranking position.

## Limitations

- Default fragmentation depth of 3 bond dissociations is insufficient for some high-energy spectra; challenge 16 peaks (m/z 122.0408-221.07309) remained unexplained until depth was raised to 5, indicating the parameter must be tuned per challenge.
- Performance depends on the relevant candidate molecule actually being present in the retrieved candidate set and on an appropriate mass tolerance; an over-restrictive tolerance (10 ppm with 0.002 Da absolute minimum) caused challenge 11 fragment peaks to go unexplained.
- Aloxistatin ranking deteriorated (42 to 58) when the extensive 5-bond setting was applied to challenge 13, so deeper fragmentation is not uniformly beneficial across all challenges.

## Evidence

- [methods] systematically fragment all candidate molecules: [evidence span withheld — anchor status: not_found]
- [methods] neutral-loss expansion: [evidence span withheld — anchor status: not_found]
- [methods] total penalty score definition: [evidence span withheld — anchor status: not_found]
- [methods] penalty-based candidate ranking: [evidence span withheld — anchor status: not_found]
- [results] challenge 16 unexplained peaks at default depth: [evidence span withheld — anchor status: not_found]
- [results] challenge 11 over-restrictive mass tolerance: [evidence span withheld — anchor status: not_found]
