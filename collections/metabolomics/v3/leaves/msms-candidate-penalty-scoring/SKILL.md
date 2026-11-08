---
name: msms-candidate-penalty-scoring
description: Use when you have MS/MS (or MSn) peak lists with a precursor elemental
  composition and need to rank candidate structures from a chemical database without
  reference spectra.
when_to_use_negative:
- The candidate structure of interest is absent from the database snapshot or excluded
  by element/mass filters — MAGMa cannot rank what it cannot retrieve (the article
  notes correct candidates must be present in the initial PubChem set).
- Reference spectra are available and library matching would suffice — this skill
  targets annotation in the absence of informative reference spectra.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0218
tools:
- name: MAGMa
  role: Runs the substructure generation, peak matching, and penalty-score ranking
    from the hierarchical peak list input; usable in command line mode or via the
    web interface.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules and refscore values used both for candidate
    retrieval (SDF) and tie-breaking in the ranking.
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:3ba80d1c1a7c9ea4e10e137e403a0038a285e211b431740480df82efeac65a63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# msms-candidate-penalty-scoring

## Summary

Rank candidate chemical structures for an MS/MS challenge by matching observed fragment m/z values against in silico generated substructures (MAGMa) and computing intensity-weighted penalty scores, as applied to the CASMI 2013 contest peak lists.

## When to use

Use when you have MS/MS (or MSn) peak lists with a precursor elemental composition and need to rank candidate structures from a chemical database without reference spectra.

## When NOT to use

- The candidate structure of interest is absent from the database snapshot or excluded by element/mass filters — MAGMa cannot rank what it cannot retrieve (the article notes correct candidates must be present in the initial PubChem set).
- Reference spectra are available and library matching would suffice — this skill targets annotation in the absence of informative reference spectra.

## Inputs

- CASMI-style MS/MS peak lists with precursor mono-isotopic peak and nested fragment hierarchy
- Elemental composition of the precursor ion (derived from accurate mass, e.g. 10 ppm / 0.002 Da window)
- Local PubChem compound database (C/H/N/O/P/S only, up to 1200 Da) with refscore annotations

## Outputs

- Ranked candidate structure list ordered by reversed penalty score (ties ordered by PubChem refscore)
- Total intensity-weighted penalty score per candidate molecule
- Mapping of observed MS/MS m/z values to best-scoring substructures, flagging non-matched peaks (penalty 10)

## How to apply

Convert the contest peak lists into MAGMa hierarchical input files, nesting MS/MS fragments in parentheses after the precursor mono-isotopic peak (composite spectra can be built by merging peaks from different collision energies, keeping the highest intensity). Retrieve candidates matching the elemental composition from a local PubChem snapshot (2 December 2013, C/H/N/O/P/S only, up to 1200 Da) using a 10 ppm mass deviation with an absolute minimum of 0.002 Da (4 ppm for specific challenges), assuming [M+H]+ or [M-H]- precursors. Generate substructures by applying up to 3 bond dissociations, expanded with single losses of hydroxyl (water) and primary amine (ammonia) groups, then match each MS/MS m/z to its best-scoring substructure, assigning a penalty of 10 to non-matched peaks. Compute each candidate's total penalty as the intensity-weighted average of substructure penalties, rank by reversed score, and break ties by PubChem refscore; reduce oversized candidate lists (e.g. challenges 1, 2, 14) with a refscore threshold of 5.

## Related tools

- **MAGMa** (Runs the substructure generation, peak matching, and penalty-score ranking from the hierarchical peak list input; usable in command line mode or via the web interface.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules and refscore values used both for candidate retrieval (SDF) and tie-breaking in the ranking.)

## Evaluation signals

- Reproducing the reported CASMI 2013 category-2 outcome: correct structure ranked first in 6 of 12 submitted solutions and outside the top 5 in 3 cases.
- Every MS/MS peak is either matched to a best-scoring substructure or explicitly assigned the non-matched penalty of 10.
- Total penalty equals the intensity-weighted average of substructure penalties for each candidate.
- Candidate sets were retrieved within the stated mass window (10 ppm, absolute minimum 0.002 Da; 4 ppm for challenges 4, 5, 6) and, for challenges 1, 2, and 14, reduced by the refscore threshold of 5.
- Equal-score candidates are consistently ordered by PubChem refscore, with tied candidates assigned their average ranking position.

## Limitations

- Too-restrictive mass tolerance (10 ppm / 0.002 Da) can leave fragment peaks unexplained even when correct tautomers are in the candidate list (challenge 11).
- The default fragmentation depth of up to 3 bond dissociations is insufficient for some molecules — several higher-energy peaks in challenge 16 could not be explained until extended to 5 dissociations.
- Performance depends on inclusion of the relevant candidate molecule, the appropriate mass tolerance, and sufficient in silico fragmentation; failing candidates must first be checked for database coverage before tuning fragmentation.

## Evidence

- [methods] penalty computation: [evidence span withheld — anchor status: not_found]
