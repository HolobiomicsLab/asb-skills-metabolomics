---
name: candidate-ranking-with-tie-breaking
description: Use when a list of candidate molecules (e.g. retrieved from PubChem within
  a 10 ppm mass window) must be ranked against an MS/MS peak list without reference
  spectra, and ties between equally scoring candidates need deterministic resolution.
when_to_use_negative:
- Informative reference spectra are available — spectral library matching is preferable
  to substructure-penalty ranking.
- The correct candidate may be absent from the candidate set (e.g. over-restrictive
  mass tolerance or missing halogen database) — ranking cannot recover a missing candidate,
  as seen for challenge 11 where the 10 ppm / 0.002 Da tolerance was too restrictive.
- The analysis target is molecular formulas only (CASMI category 1) rather than structure
  ranking in category 2.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: MAGMa
  role: Performs systematic in silico fragmentation of candidates, matches peaks to
    substructures, computes intensity-weighted penalty scores, and reverses scores
    into rankings; runs in command line mode or via the emetabolomics web interface
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules and of the refscore used to order equally scoring
    candidates (threshold 5 used to reduce candidate lists)
- name: emetabolomics web interface
  role: Web interface (www.emetabolomics.org/magma) to launch MAGMa jobs and view
    ranking results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:aec9ec3a32b851eb290b25e9982d3d31ccd4b4f44ef8a203c0e55a91eed51a1b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# candidate-ranking-with-tie-breaking

## Summary

Ranks candidate molecular structures by a MAGMa penalty score computed from intensity-weighted matches between observed MS/MS peaks and in silico generated substructures, resolving ties via PubChem refscore and assigning tied candidates their average ranking position.

## When to use

Use when a list of candidate molecules (e.g. retrieved from PubChem within a 10 ppm mass window) must be ranked against an MS/MS peak list without reference spectra, and ties between equally scoring candidates need deterministic resolution.

## When NOT to use

- Informative reference spectra are available — spectral library matching is preferable to substructure-penalty ranking.
- The correct candidate may be absent from the candidate set (e.g. over-restrictive mass tolerance or missing halogen database) — ranking cannot recover a missing candidate, as seen for challenge 11 where the 10 ppm / 0.002 Da tolerance was too restrictive.
- The analysis target is molecular formulas only (CASMI category 1) rather than structure ranking in category 2.

## Inputs

- MAGMa hierarchical input files nesting MS/MS fragment peaks in parentheses after the precursor mono-isotopic peak
- Candidate molecule set retrieved from local PubChem (2 December 2013 snapshot; CHNOPS-only up to 1200 Da, or halogenated supplement) within 10 ppm / 0.002 Da minimum mass deviation
- Composite MS/MS spectra merging peaks from different collision energies

## Outputs

- Ranked list of candidate structures ordered by reversed penalty score, with tied candidates assigned average ranking positions
- Per-candidate penalty scores from substructure annotation (e.g. written to retrospective_rankings.csv)

## How to apply

Run MAGMa's systematic in silico fragmentation on each candidate (up to 3 bond dissociations by default, or 5 for retrospective/extended analysis, expanding the substructure set with single losses of hydroxyl and primary amine groups) and match MS/MS m/z values to the best-scoring substructures. Compute each candidate's total penalty score as the intensity-weighted average of substructure penalties, where a substructure penalty is the sum of penalty values for all dissociated bonds and non-matched peaks incur a penalty of 10. Rank candidates by the reversed penalty score so lower total penalty corresponds to a better (numerically lower) rank. For equally scoring candidates, order them on the basis of their PubChem refscore; candidates that remain tied share the average of their ranking positions, which is how aloxistatin's rank 42 (later 58 with 5-bond fragmentation in challenge 13) was reported.

## Related tools

- **MAGMa** (Performs systematic in silico fragmentation of candidates, matches peaks to substructures, computes intensity-weighted penalty scores, and reverses scores into rankings; runs in command line mode or via the emetabolomics web interface) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules and of the refscore used to order equally scoring candidates (threshold 5 used to reduce candidate lists))
- **emetabolomics web interface** (Web interface (www.emetabolomics.org/magma) to launch MAGMa jobs and view ranking results)

## Evaluation signals

- Correct candidates are recovered at expected ranks, e.g. 2-(perfluorooctyl)ethanol at rank 1 in challenge 15 and ofloxacin at rank 126 in challenge 16 under 5-bond fragmentation.
- Every candidate has a finite penalty score; non-matched peaks contribute exactly 10 each to the intensity-weighted average.
- Equally scoring candidates are ordered by PubChem refscore and share an averaged ranking position, not an arbitrary index.
- Reproduced rankings match the published table at http://www.emetabolomics.org/results_CASMI2013.html (in 6 of 12 submitted solutions the correct candidate ranked first).
- Candidate sets were filtered to consistent sizes via refscore thresholds (<1000, <500, <200 molecules).

## Limitations

- Raising fragmentation depth from 3 to 5 bond dissociations can worsen rankings — aloxistatin deteriorated from rank 42 to 58 in challenge 13 — so extended fragmentation is not uniformly beneficial.
- Too-restrictive mass tolerance (10 ppm with 0.002 Da minimum) can leave fragment peaks unexplained even when correct tautomers are present in the candidate list (challenge 11).
- Default 3-bond dissociation is insufficient for some high-energy fragments (e.g. challenge 16 peaks m/z=122.0408 through 221.07309), requiring retrospective deeper fragmentation.
- Ranking is only as good as the candidate set: successful application required inclusion of the relevant candidate molecules, appropriate mass tolerance, and sufficient in silico fragmentation.

## Evidence

- [abstract] penalty-based ranking: "The MS/MS peaks of each challenge were matched with in silico generated substructures of candidate molecules from PubChem, resulting in penalty scores that were used for candidate ranking."
- [methods] total score computation: [evidence span withheld — anchor status: not_found]
- [methods] tie-breaking rule: "Equally scoring candidates were further ordered on the basis of their Pub Chem refscore"
- [methods] mass tolerance: "The allowed mass deviation was set to 10 ppm, with an absolute minimum of 0.002 Da"
- [results] extended fragmentation effect: [evidence span withheld — anchor status: not_found]
- [readme] software availability: "MAGMa is a abbreviation for 'Ms Annotation based on in silico Generated Metabolites'."
