---
name: in-silico-fragmentation-scoring
description: Use when MS/MS (or MSn) peak lists must be annotated against candidate
  structures retrieved by precursor mass, and candidates need to be ranked by spectral
  explanation without reference spectra.
when_to_use_negative:
- Reference spectra are available and a rule-based/commercial tool (e.g. Mass Frontier)
  or spectral-library match is the appropriate route
- The task is pure formula enumeration with no structural candidates to score against
  fragment peaks
- Peaks require fragmentation beyond 3 bond dissociations or multistage MSn (n>2)
  that the CASMI MS/MS workflow does not cover
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MAGMa
  role: Runs in-silico fragmentation of candidate molecules, substructure-peak matching
    and penalty-based candidate ranking for MS/MS data
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules retrieved by precursor mass (SDF), providing
    refscore used to order equally scoring candidates
- name: emetabolomics web interface
  role: Web front-end to submit MAGMa jobs and view ranked annotation results
  repo: http://www.emetabolomics.org/results_CASMI2013.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:620c14f70f7ad214e68ba21deb2c93451d4a6eb00eb91daa585e02bd952fb9cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# in-silico-fragmentation-scoring

## Summary

Systematically fragments candidate molecules in silico (substructures from up to 3 bond dissociations plus water/ammonia losses), matches observed MS/MS peaks against those substructures, and computes intensity-weighted penalty scores to rank candidates by how well they explain the spectrum.

## When to use

Use when MS/MS (or MSn) peak lists must be annotated against candidate structures retrieved by precursor mass, and candidates need to be ranked by spectral explanation without reference spectra.

## When NOT to use

- Reference spectra are available and a rule-based/commercial tool (e.g. Mass Frontier) or spectral-library match is the appropriate route
- The task is pure formula enumeration with no structural candidates to score against fragment peaks
- Peaks require fragmentation beyond 3 bond dissociations or multistage MSn (n>2) that the CASMI MS/MS workflow does not cover

## Inputs

- MS/MS (or multistage MSn) peak lists from CASMI-style challenges, including spectra at different collision energies
- Candidate molecule structures (e.g. SDF retrieved from PubChem by precursor mass)
- Precursor mass and allowed mass deviation (ppm / absolute Da)

## Outputs

- Substructure-to-peak match assignments per candidate
- Penalty score per candidate (intensity-weighted average over matched MS/MS peaks)
- Ranked candidate list with formulas/structures for submission or reporting

## How to apply

Convert peak lists to MAGMa hierarchical input (precursor peak with fragment peaks nested in parentheses, merging collision energies into composite spectra and keeping the highest intensity per ion). Retrieve candidates by precursor mass within the allowed deviation (10 ppm, absolute minimum 0.002 Da; 4 ppm for challenges 4-6), e.g. from a local PubChem snapshot. Generate substructures with up to 3 bond dissociations, expanded with single losses of hydroxyl or primary amine groups, then match each MS/MS m/z with the best scoring substructure. Score each candidate as an intensity-weighted average of substructure penalties, with unmatched peaks penalized at 10, and rank in reversed penalty order, breaking ties by PubChem refscore.

## Related tools

- **MAGMa** (Runs in-silico fragmentation of candidate molecules, substructure-peak matching and penalty-based candidate ranking for MS/MS data) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules retrieved by precursor mass (SDF), providing refscore used to order equally scoring candidates)
- **emetabolomics web interface** (Web front-end to submit MAGMa jobs and view ranked annotation results) — http://www.emetabolomics.org/results_CASMI2013.html

## Examples

```
magma --input challenge.sdf --spectra peaks.magma --mass-tolerance 10ppm --max-dissociations 3
```

## Evaluation signals

- Top-ranked candidate's formula matches the known/expected formula (in CASMI category 1, all 8 top-ranked formulas were correct)
- Unmatched fragment peaks should be few; systematically unexplained m/z values (e.g. challenge 16 peaks up to m/z=221.07309) signal insufficient fragmentation depth or too-restrictive mass tolerance
- Candidate list sizes after refscore filtering should be in the intended ranges (e.g. <1000, <500, <200) before scoring
- Equally scoring candidates are consistently ordered by refscore, so rankings are reproducible
- Composite spectra preserve the highest intensity when an ion appears at multiple collision energies

## Limitations

- Default in-silico fragmentation depth (3 bond dissociations) was insufficient for challenge 16, leaving several higher-energy peaks unexplained
- Overly restrictive mass tolerance (10 ppm / 0.002 Da) left challenge 11 fragment peaks unexplained although correct tautomers were present
- Performance depends on the relevant candidate being included in the retrieved set; missing candidates cause annotation failure
- MAGMa was developed for MSn with n>2; MS/MS performance is comparable to other methods but not its original design point
- Large candidate sets require refscore pre-filtering, which may drop the correct molecule

## Evidence

- [methods] A total penalty score for a given candidate is calculated as an intensity weighted average of the substructure penalties: [evidence span withheld — anchor status: not_found]
- [methods] This set of substructures was expanded with single losses of hydroxyl groups (representing neutral loss of water) or primary amine groups (representing neutral loss of ammonia).: [evidence span withheld — anchor status: not_found]
- [results] The input data was constructed by generating composite spectra of the MS/MS spectra obtained at the different collision energies: [evidence span withheld — anchor status: not_found]
