---
name: halogenated-compound-database-retrieval
description: Use when a CASMI-style MS/MS challenge has a precursor elemental composition
  containing halogens (F, Cl, Br) and the default CHNOPS-only PubChem candidate database
  would miss the correct structure.
when_to_use_negative:
- The precursor composition contains only C, H, N, O, P, S — the standard PubChem
  subset suffices and the halogenated supplement is unnecessary
- Reference spectra are available for the unknown — MAGMa annotation (and hence this
  retrieval) targets annotation in the absence of informative reference spectra
- The mass accuracy of the instrument is worse than the 10 ppm / 0.002 Da assumed
  maximum error — the challenge-11 retrospective showed an overly restrictive tolerance
  can leave correct candidates unexplained
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MAGMa
  role: Retrieves candidate molecules from the local PubChem database (including the
    halogenated supplement) and processes them for in silico substructure-based annotation
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate structures, retrieved in SDF format; halogenated compounds
    come from an added local halogenated PubChem database
- name: emetabolomics web interface
  role: Alternative processing entry point (www.emetabolomics.org/magma) for running
    the retrieval-then-annotation workflow
- name: pubchem (MAGMa subproject)
  role: Repository subapplication for processing the PubChem database used to find
    mass candidates
  repo: https://github.com/NLeSC/MAGMa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:183ba4942a4468302c1bca9c8c47e11567ca6c076a3a128a108f2fe3236126f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  - pubchem (MAGMa subproject)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# halogenated-compound-database-retrieval

## Summary

Retrieves candidate molecule structures containing halogens from an added local halogenated PubChem database (snapshot of all PubChem compounds, e.g. 2 December 2013) so that challenges involving fluorinated/chlorinated unknowns — like CASMI 2013 challenges 15 and 16 — can be annotated by MAGMa.

## When to use

Use when a CASMI-style MS/MS challenge has a precursor elemental composition containing halogens (F, Cl, Br) and the default CHNOPS-only PubChem candidate database would miss the correct structure.

## When NOT to use

- The precursor composition contains only C, H, N, O, P, S — the standard PubChem subset suffices and the halogenated supplement is unnecessary
- Reference spectra are available for the unknown — MAGMa annotation (and hence this retrieval) targets annotation in the absence of informative reference spectra
- The mass accuracy of the instrument is worse than the 10 ppm / 0.002 Da assumed maximum error — the challenge-11 retrospective showed an overly restrictive tolerance can leave correct candidates unexplained

## Inputs

- Precursor elemental composition / precursor monoisotopic m/z from a CASMI challenge peak list
- Local PubChem compound database snapshot (2 December 2013) with added halogenated supplement
- Mass tolerance parameters (10 ppm, absolute minimum 0.002 Da)

## Outputs

- SDF-format candidate molecule list containing halogenated structures matching the precursor within tolerance
- Candidate set fed into MAGMa in silico fragmentation for penalty-score ranking

## How to apply

Build a local PubChem database snapshot (e.g. all PubChem compounds on 2 December 2013) using the MAGMa repository's 'pubchem' processing application, and add a halogenated supplement covering compounds beyond the C, H, N, O, P, S element set. For the given elemental composition, retrieve candidates allowing a mass deviation of 10 ppm with an absolute minimum of 0.002 Da, fetching structures in SDF format (directly from the online PubChem service for the standard set, or from the added local halogenated database for challenges 15 and 16). Retrieve candidates before running MAGMa's in silico fragmentation, since successful annotation required inclusion of the relevant candidate molecules in the candidate set. If large candidate lists result, reduce them with the PubChem refscore threshold approach (stepwise raising the threshold to <1000, <500, and <200 molecules).

## Related tools

- **MAGMa** (Retrieves candidate molecules from the local PubChem database (including the halogenated supplement) and processes them for in silico substructure-based annotation) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate structures, retrieved in SDF format; halogenated compounds come from an added local halogenated PubChem database)
- **emetabolomics web interface** (Alternative processing entry point (www.emetabolomics.org/magma) for running the retrieval-then-annotation workflow)
- **pubchem (MAGMa subproject)** (Repository subapplication for processing the PubChem database used to find mass candidates) — https://github.com/NLeSC/MAGMa

## Evaluation signals

- Every retrieved candidate's formula-derived exact mass falls within 10 ppm of the precursor (and at least 0.002 Da absolute)
- Known correct structures for halogenated challenges are recovered: 2-(perfluorooctyl)ethanol ranked 1st for challenge 15 and ofloxacin ranked 126th for challenge 16 under extended (5-bond) fragmentation
- Retrieved structures are in SDF format and loadable by the MAGMa 'job' calculation
- Retrieved candidates are input as PubChem refscore-annotated records so equally scoring candidates can be ordered by refscore
- No correct candidate is lost to filtering: challenge 11 showed a too-restrictive tolerance (0.002 Da) can exclude the correct tautomers, so verify tolerance settings against the challenge spec

## Limitations

- A too-restrictive mass tolerance (10 ppm / 0.002 Da) can cause correct candidates to be misinterpreted or excluded (challenge 11 retrospective)
- No explicit software version, versioned release, or commit hash for MAGMa or the halogenated database supplement is stated in the article
- Correct candidates must be present in the database snapshot; absence of the relevant candidate molecules cannot be fixed by fragmentation or scoring

## Evidence

- [results] Correct candidate presence in PubChem set: [evidence span withheld — anchor status: not_found]
- [readme] PubChem processing subproject in repository: [evidence span withheld — anchor status: external_doc]
