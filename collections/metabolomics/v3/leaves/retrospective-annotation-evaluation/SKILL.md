---
name: retrospective-annotation-evaluation
description: Use when initial MAGMa annotation rankings leave specific MS/MS peaks
  unexplained or correct candidates rank poorly, and you need to retrospectively test
  whether a more permissive fragmentation depth or mass tolerance resolves the miss.
when_to_use_negative:
- When the goal is a fresh de novo annotation for a new sample rather than diagnosing
  why an existing MAGMa run ranked a known candidate poorly — use a standard MAGMa
  run instead
- When the candidate of interest is absent from the PubChem snapshot used for retrieval,
  since the retrospective rerun can only reshuffle molecules already in the candidate
  list
- When the input is MSn multistage fragmentation data with n>2 rather than simple
  MS/MS peak lists, as the hierarchical nesting described here targets MS/MS spectral
  trees
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MAGMa
  role: Executes the systematic in silico fragmentation of candidate molecules and
    computes penalty-score rankings; run in command-line mode or via the web interface
    for the retrospective rerun.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules retrieved from a local database snapshot to
    build the candidate set that is rescored with extended fragmentation settings.
- name: emetabolomics web interface
  role: Alternative entry point for running MAGMa jobs and inspecting retrospective
    results.
  repo: http://www.emetabolomics.org/results_CASMI2013.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:b205cc6675959082954589deb24e02600e4aae9326693713596c3ca9c7fa69a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# retrospective-annotation-evaluation

## Summary

Rerun MAGMa's in silico fragmentation with relaxed or extended parameters (e.g. up to 5 bond dissociations instead of 3, or looser mass tolerance) after a contest or benchmark round, to explain previously unmatched MS/MS peaks and diagnose ranking failures across challenges.

## When to use

Use when initial MAGMa annotation rankings leave specific MS/MS peaks unexplained or correct candidates rank poorly, and you need to retrospectively test whether a more permissive fragmentation depth or mass tolerance resolves the miss.

## When NOT to use

- When the goal is a fresh de novo annotation for a new sample rather than diagnosing why an existing MAGMa run ranked a known candidate poorly — use a standard MAGMa run instead
- When the candidate of interest is absent from the PubChem snapshot used for retrieval, since the retrospective rerun can only reshuffle molecules already in the candidate list
- When the input is MSn multistage fragmentation data with n>2 rather than simple MS/MS peak lists, as the hierarchical nesting described here targets MS/MS spectral trees

## Inputs

- CASMI contest MS/MS peak lists (composite spectra from different collision energies, merged by taking the highest intensity for ions present at both energies)
- Local PubChem candidate database (all compounds as of 2 December 2013, restricted to C, H, N, O, P, S for challenge 13; halogenated supplement for challenges 15 and 16)
- Previously submitted ranking results for comparison

## Outputs

- Revised candidate penalty scores and reversed rankings per challenge
- Identification of previously unexplained higher-energy m/z peaks (e.g. challenge 16 peaks at m/z 122.0408–221.07309)
- Quantified rank shifts of correct candidates between original and extended-parameter runs

## How to apply

Convert the CASMI contest peak lists into MAGMa hierarchical input files, nesting MS/MS fragment peaks in parentheses after the precursor mono-isotopic peak. Retrieve candidates from the local PubChem database (10 ppm mass deviation, absolute minimum 0.002 Da; 4 ppm for challenges 4–6) and run MAGMa with the extended dissociation depth of up to 5 bonds, expanding the substructure set with single losses of hydroxyl (water) or primary amine (ammonia) groups. Match each m/z value with the best-scoring substructure and compute the total penalty as an intensity-weighted average, penalising non-matched peaks with 10. Rank by reversed penalty score, tie-breaking on the PubChem refscore. Compare the new rankings against the original submission — e.g. aloxistatin moved from 42 to 58 in challenge 13, 2-(perfluorooctyl)ethanol rose to rank 1 in challenge 15, and ofloxacin's higher-energy peaks (m/z 122.0408, 178.03072, 179.03857, 193.04172, 194.02574, 199.05114, 205.04175, 219.05744, 221.07309) were explained — and record the shift as the diagnostic outcome.

## Related tools

- **MAGMa** (Executes the systematic in silico fragmentation of candidate molecules and computes penalty-score rankings; run in command-line mode or via the web interface for the retrospective rerun.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules retrieved from a local database snapshot to build the candidate set that is rescored with extended fragmentation settings.)
- **emetabolomics web interface** (Alternative entry point for running MAGMa jobs and inspecting retrospective results.) — http://www.emetabolomics.org/results_CASMI2013.html

## Examples

```
magma --input challenge16.magm --pubchem local_pubchem.db --bond-dissociations 5 --neutral-losses water,ammonia --output retrospective_rankings.csv
```

## Evaluation signals

- Reproduced retrospective rankings match the reported outcomes: aloxistatin at rank 58 in challenge 13 (deteriorated from 42), 2-(perfluorooctyl)ethanol at rank 1 in challenge 15, and ofloxacin at rank 126 in challenge 16
- All higher-energy peaks listed for challenge 16 (m/z 122.0408, 178.03072, 179.03857, 193.04172, 194.02574, 199.05114, 205.04175, 219.05744, 221.07309) are matched to substructures under the 5-bond setting
- Candidate sets are stepwise reduced by refscore thresholds to <1000, <500, and <200 molecules before scoring
- Mass tolerance for candidate retrieval stays at 10 ppm with an absolute minimum of 0.002 Da (4 ppm for challenges 4–6), and loosening this tolerance is only applied when challenge 11's peaks remain unexplained
- Tied candidates are ordered by PubChem refscore and assigned their average ranking position

## Limitations

- Extended fragmentation to 5 bond dissociations can deteriorate ranking of the correct candidate (aloxistatin slipped from rank 42 to 58 in challenge 13), so depth increases are not uniformly beneficial
- Retrospective analysis depends on the relevant candidate being present in the initial PubChem candidate set; if it is missing, re-scoring cannot recover it
- A too-restrictive mass tolerance (10 ppm with 0.002 Da minimum) was identified as the likely cause of unexplained peaks in challenge 11, indicating that tolerance settings must be challenged during evaluation

## Evidence

- [results] Retrospective extended fragmentation depth diagnosis: [evidence span withheld — anchor status: not_found]
- [results] Mass tolerance mis-specification as retrospective diagnosis: [evidence span withheld — anchor status: not_found]
- [results] Unexplained higher-energy peaks motivating the rerun: [evidence span withheld — anchor status: not_found]
- [readme] MAGMa runs from repository (readme): [evidence span withheld — anchor status: external_doc]
- [results] Composite spectrum construction: [evidence span withheld — anchor status: not_found]
