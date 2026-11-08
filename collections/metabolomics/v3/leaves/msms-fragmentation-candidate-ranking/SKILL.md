---
name: msms-fragmentation-candidate-ranking
description: Use when a challenge provides MS/MS peak lists (possibly merged across
  collision energies) with a known elemental composition, and candidate structures
  must be retrieved from PubChem and ranked without reference spectra.
when_to_use_negative:
- The unknown's high-energy fragment peaks require more than 3 bond dissociations
  to explain (as in challenge 16, where peaks such as m/z=122.0408 and m/z=221.07309
  remained unexplained) — extend the fragmentation level instead of ranking with defaults.
- A reference spectral library is available and library matching is sufficient; MAGMa's
  substructure annotation is designed for the absence of informative reference spectra.
- The correct compound is likely to carry a very low PubChem refscore — stepwise refscore
  filtering can eliminate it, as happened for challenge 3's N2-acetyl glutaminyl leucinamide
  (refscore 1) once lists dropped below 200 molecules.
tools:
- name: MAGMa
  role: Performs in silico fragmentation of candidate molecules, substructure-to-peak
    matching, and penalty-score-based candidate ranking; run in command line mode
    or via the emetabolomics web interface.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules with the provided elemental composition (SDF
    retrieval or local snapshot) and of the refscore used for tie-breaking and threshold
    filtering.
- name: emetabolomics web interface
  role: Alternative entry point to run MAGMa jobs and view ranked results when not
    using command line mode.
  repo: http://www.emetabolomics.org/magma
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:991f0e8903568d9b7d97cc02c7d1c11fc5c17ad6d914c78ef1c62def6a50fafc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-fragmentation-candidate-ranking

## Summary

Rank candidate chemical structures for an unknown MS/MS spectrum by matching observed peaks against in silico generated substructures (up to 3 bond dissociations) of PubChem candidates, yielding intensity-weighted penalty scores, with PubChem refscore used to break score ties and to stepwise filter oversized candidate lists.

## When to use

Use when a challenge provides MS/MS peak lists (possibly merged across collision energies) with a known elemental composition, and candidate structures must be retrieved from PubChem and ranked without reference spectra.

## When NOT to use

- The unknown's high-energy fragment peaks require more than 3 bond dissociations to explain (as in challenge 16, where peaks such as m/z=122.0408 and m/z=221.07309 remained unexplained) — extend the fragmentation level instead of ranking with defaults.
- A reference spectral library is available and library matching is sufficient; MAGMa's substructure annotation is designed for the absence of informative reference spectra.
- The correct compound is likely to carry a very low PubChem refscore — stepwise refscore filtering can eliminate it, as happened for challenge 3's N2-acetyl glutaminyl leucinamide (refscore 1) once lists dropped below 200 molecules.

## Inputs

- CASMI challenge MS/MS peak lists (converted to MAGMa input format), optionally as composite spectra across collision energies
- Elemental composition of the unknown compound
- Local PubChem compound database snapshot (2 December 2013) or online PubChem SDF retrieval
- Per-candidate PubChem refscore values for tie-breaking and filtering

## Outputs

- Ranked candidate structure list per challenge (reversed ranking value based on penalty score)
- Substructure penalty scores per candidate
- Tabulation of correct-candidate absolute rank at refscore thresholds <1000, <500, <200 candidates (refscore_filtering_table.csv)

## How to apply

Convert the CASMI contest peak lists to MAGMa input format, optionally building composite spectra by merging MS/MS peaks from different collision energies and taking the highest intensity for ions present at both energies. Retrieve candidate molecules matching the provided elemental composition from a local PubChem database snapshot (2 December 2013) within a 10 ppm mass deviation (absolute minimum 0.002 Da; tighten to 4 ppm for challenges 4, 5, and 6). MAGMa then fragments each candidate systematically, generating substructures by up to 3 bond dissociations (expanded with single losses of hydroxyl or primary amine groups for water/ammonia neutral losses), matches peak m/z values with the best-scoring substructures, and computes a total penalty score as an intensity-weighted average of substructure penalties. Rank candidates by reversed penalty score and order equally scoring candidates by their PubChem refscore; when initial candidate lists are very large (e.g. challenges 1, 2, and 14), apply a refscore threshold (value 5 in the initial submission) or stepwise raise the threshold until sets are reduced to <1000, <500, and <200 molecules, verifying that the correct compound's absolute rank improves as the list shrinks.

## Related tools

- **MAGMa** (Performs in silico fragmentation of candidate molecules, substructure-to-peak matching, and penalty-score-based candidate ranking; run in command line mode or via the emetabolomics web interface.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules with the provided elemental composition (SDF retrieval or local snapshot) and of the refscore used for tie-breaking and threshold filtering.)
- **emetabolomics web interface** (Alternative entry point to run MAGMa jobs and view ranked results when not using command line mode.) — http://www.emetabolomics.org/magma

## Evaluation signals

- In six of twelve submitted category 2 solutions the correct candidate ranked first, with only three outside the top 5 — a new run should broadly reproduce this distribution.
- Stepwise refscore filtering (<1000, <500, <200 candidates) must keep the correct compound in almost all reduced lists and improve its absolute rank as list size decreases.
- The challenge 3 counter-example (N2-acetyl glutaminyl leucinamide, refscore 1) must be present as the single loss when the list is reduced below 200 molecules.
- All candidate retrievals must respect the mass deviation window: 10 ppm with absolute minimum 0.002 Da, or 4 ppm for challenges 4, 5, and 6.
- All top-ranked molecular formulas submitted in category 1 should be correct; check the combined table is written to refscore_filtering_table.csv.

## Limitations

- Successful ranking requires the relevant candidate molecule to be present in the PubChem candidate set, an appropriate mass tolerance, and sufficient in silico fragmentation.
- The assumed maximum error of 10 ppm and 0.002 Da was too restrictive for challenge 11, leaving fragment peaks unexplained even though the correct tautomers were in the candidate list.
- Default in silico fragmentation (up to 3 bond dissociations) was insufficient for challenge 16, where several higher-energy peaks could not be explained.
- Refscore tie-breaking and threshold filtering are heuristic: low-refscore correct compounds (e.g. refscore 1) can be lost when lists are aggressively reduced.

## Evidence

- [methods] A total penalty score for a given candidate is calculated as an intensity weighted average of the substructure penalties: [evidence span withheld — anchor status: not_found]
- [results] For each challenge we stepwise raised the threshold on the refscore until the candidates sets were reduced to <1000, <500, and <200 molecules.: [evidence span withheld — anchor status: not_found]
