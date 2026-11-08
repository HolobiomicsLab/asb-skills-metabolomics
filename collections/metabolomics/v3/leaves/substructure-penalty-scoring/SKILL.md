---
name: substructure-penalty-scoring
description: Use when annotating MS/MS or MSn spectra where no informative reference
  spectra exist and candidate structures (e.g. from PubChem) must be ranked by how
  well they explain observed fragment peaks.
when_to_use_negative:
- Informative reference spectra are available for library matching — this skill is
  designed for the absence of reference spectra.
- The candidate structure contains fragmentations requiring more than 3 bond dissociations
  — in challenge 16 several higher-energy peaks (e.g. m/z=122.0408, 221.07309) could
  not be explained with up to 3 bond dissociations.
- The task is reducing an already-scored candidate list by database quality rather
  than structural explanation — use refscore threshold filtering (<1000/<500/<200
  molecules) instead.
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: MAGMa
  role: Performs systematic in silico fragmentation of candidates, substructure matching,
    penalty scoring, and candidate ranking; run in command line mode or via web interface
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate structures in SDF format, retrieved by elemental composition;
    refscore used for tie-breaking among equally scoring candidates
- name: emetabolomics web interface
  role: Web front-end (www.emetabolomics.org/magma) to run MAGMa jobs and view ranked
    annotation results
  repo: http://www.emetabolomics.org/magma
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:7f967e306cca19c76a5f2158b80b854cae5953e22e6ceda542d5cab440954898
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# substructure-penalty-scoring

## Summary

Ranks candidate chemical structures by matching observed MS/MS m/z values against in silico generated substructures (up to 3 bond dissociations) of each candidate, producing intensity-weighted penalty scores for candidate ranking without reference spectra.

## When to use

Use when annotating MS/MS or MSn spectra where no informative reference spectra exist and candidate structures (e.g. from PubChem) must be ranked by how well they explain observed fragment peaks.

## When NOT to use

- Informative reference spectra are available for library matching — this skill is designed for the absence of reference spectra.
- The candidate structure contains fragmentations requiring more than 3 bond dissociations — in challenge 16 several higher-energy peaks (e.g. m/z=122.0408, 221.07309) could not be explained with up to 3 bond dissociations.
- The task is reducing an already-scored candidate list by database quality rather than structural explanation — use refscore threshold filtering (<1000/<500/<200 molecules) instead.

## Inputs

- MS/MS peak lists from CASMI contest challenges (composite spectra across collision energies)
- Candidate molecular structures retrieved from PubChem in SDF format by elemental composition
- Molecular formula assignments (category 1 results) per challenge

## Outputs

- Per-candidate substructure penalty scores and total intensity-weighted penalty scores
- Ranked candidate lists per challenge (reversed ranking values), with refscore-based tie-breaking
- MAGMa annotation results page / submitted CASMI solution files

## How to apply

Convert the CASMI contest peak lists to MAGMa input files, merging MS/MS peaks from different collision energies into composite spectra (taking the highest intensity for ions present at both energies). Retrieve candidates with the provided elemental composition from PubChem (local database of all PubChem compounds on 2 December 2013), then systematically fragment each candidate applying up to 3 bond dissociations, expanding substructures with single losses of hydroxyl groups (water loss) or primary amine groups (ammonia loss). Match m/z values of the MS/MS peaks with the best scoring substructures; a substructure penalty is the sum of penalty values for all dissociated bonds. Compute the total penalty score per candidate as an intensity-weighted average of the substructure penalties, rank candidates by reversed penalty score, and order equally scoring candidates by PubChem refscore. Use a mass deviation of 10 ppm with an absolute minimum of 0.002 Da (4 ppm tolerance for challenges 4, 5, and 6).

## Related tools

- **MAGMa** (Performs systematic in silico fragmentation of candidates, substructure matching, penalty scoring, and candidate ranking; run in command line mode or via web interface) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate structures in SDF format, retrieved by elemental composition; refscore used for tie-breaking among equally scoring candidates)
- **emetabolomics web interface** (Web front-end (www.emetabolomics.org/magma) to run MAGMa jobs and view ranked annotation results) — http://www.emetabolomics.org/magma

## Evaluation signals

- In 6 of 12 submitted category 2 solutions the correct chemical structure obtained the best score; 3 molecules ranked outside the top 5 — a reproduction should land in this range.
- All top-ranked molecular formulas submitted in category 1 should be correct.
- Mass tolerance settings check: 10 ppm with 0.002 Da absolute minimum (4 ppm for challenges 4, 5, 6) — deviations cause misses like challenge 11's unexplained tautomers.
- Equally scoring candidates must be ordered by PubChem refscore, not arbitrarily.
- Per-peak explanation check: every matched m/z should trace to a best-scoring substructure generated by ≤3 bond dissociations (plus water/ammonia neutral losses); unexplained peaks like challenge 16's m/z=205.04175 flag insufficient fragmentation.

## Limitations

- Default in silico fragmentation (up to 3 bond dissociations) was insufficient for challenge 16, where several higher-energy peaks could not be explained.
- Too-restrictive mass tolerance (10 ppm / 0.002 Da) caused challenge 11's fragment peaks to be initially unexplained even though correct tautomers were in the candidate list.
- Performance depends on inclusion of the relevant candidate molecules in the retrieved PubChem candidate set; successful application requires appropriate mass tolerance and sufficient in silico fragmentation.

## Evidence

- [methods] total penalty score definition: [evidence span withheld — anchor status: not_found]
