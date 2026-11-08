---
name: elemental-composition-database-filtering
description: Use when MS/MS peak lists fail to yield the correct structure from a
  composition-restricted candidate database, e.g. isotope ratios indicate Cl and the
  correct halogenated molecule is absent from candidate sets.
when_to_use_negative:
- The elemental composition is already known and provided — in that case candidates
  were retrieved directly from the online PubChem service in SDF format rather than
  by composition-filtered database search.
- The default C, H, N, O, P, S composition filter already yields the correct candidate
  — enlarging the database with halogens only increases the candidate pool and subsequent
  refscore filtering burden.
- The challenge requires a narrower mass tolerance (e.g. 4 ppm for challenges 4, 5,
  and 6) — this skill uses the default 10 ppm / 0.002 Da retrieval window.
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: MAGMa
  role: Performs candidate retrieval from the local PubChem-derived database using
    the composition filter and mass tolerance, in command line mode or via the web
    interface.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of the local compound database snapshot (2 December 2013) and online
    source of SDF-format candidate records queried by mono-isotopic mass.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:6e38e379c35b87a77ffc0568f6774dbea5032c4c99c4612a1092c91f4b49ccac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# elemental-composition-database-filtering

## Summary

Extends a local MAGMa compound database built from a PubChem snapshot (2 December 2013) by relaxing the elemental-composition filter to include halogen atoms, so that candidate molecules containing Cl or F can be retrieved by accurate-mass search for challenges where the C,H,N,O,P,S-only database missed the correct structure.

## When to use

Use when MS/MS peak lists fail to yield the correct structure from a composition-restricted candidate database, e.g. isotope ratios indicate Cl and the correct halogenated molecule is absent from candidate sets.

## When NOT to use

- The elemental composition is already known and provided — in that case candidates were retrieved directly from the online PubChem service in SDF format rather than by composition-filtered database search.
- The default C, H, N, O, P, S composition filter already yields the correct candidate — enlarging the database with halogens only increases the candidate pool and subsequent refscore filtering burden.
- The challenge requires a narrower mass tolerance (e.g. 4 ppm for challenges 4, 5, and 6) — this skill uses the default 10 ppm / 0.002 Da retrieval window.

## Inputs

- Local MAGMa compound database derived from the PubChem snapshot of 2 December 2013 (mass cutoff 1200 Da)
- Elemental composition filter previously limited to C, H, N, O, P, and S
- Peak masses from CASMI 2013 challenge MS/MS datasets (e.g. challenges 9, 15, 16)
- SDF-format candidate records retrieved from the online PubChem service

## Outputs

- Halogen-augmented candidate lists for challenges 9, 15, and 16 written to halogen_candidates.csv
- Candidate sets containing the correct halogenated structures (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin)

## How to apply

Build the local MAGMa compound database from all PubChem compounds on 2 December 2013 with the 1200 Da mass cutoff, then add halogen elements to the composition filter that previously restricted compounds to C, H, N, O, P, and S. For challenges where the MS file's isotope ratios reveal chlorine (e.g. challenge 9), optionally retrieve candidates via an online PubChem query on the mono-isotopic mass and handle the returned SDF-format records as retrieval input. For challenges 15 and 16, retrieve candidates from the halogen-augmented local database using the default mass tolerance of 10 ppm with an absolute minimum of 0.002 Da (whichever is larger for a given peak mass). Finally, verify that the known halogenated targets (chlorpyrifos for challenge 9, 2-(perfluorooctyl)ethanol for challenge 15, ofloxacin for challenge 16) are present in the candidate sets and write the lists for challenges 9, 15, and 16 to halogen_candidates.csv. The rationale is that successful MAGMa annotation required inclusion of the relevant candidate molecules; without the halogen extension the correct structures for challenges 15 and 16 could not be retrieved at all.

## Related tools

- **MAGMa** (Performs candidate retrieval from the local PubChem-derived database using the composition filter and mass tolerance, in command line mode or via the web interface.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of the local compound database snapshot (2 December 2013) and online source of SDF-format candidate records queried by mono-isotopic mass.)

## Evaluation signals

- Each candidate set for challenges 9, 15, and 16 is non-empty and contains the correct halogenated structure (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin).
- Every retrieved candidate's monoisotopic mass falls within max(10 ppm, 0.002 Da) of the queried peak mass.
- Retrieved compositions contain only allowed elements (C, H, N, O, P, S plus halogens) and fall under the 1200 Da mass cutoff.
- The candidate lists for challenges 9, 15, and 16 are successfully written to halogen_candidates.csv.
- Retrospective MAGMa processing with the augmented candidates produces satisfactory entries for challenges 15 and 16, consistent with the paper's report.

## Limitations

- The composition filter extension was only made after the CASMI deadline, so the original submissions for challenges 15 and 16 missed the correct structures; this skill is retrospective for those challenges.
- The online PubChem retrieval path handles SDF-format records only and was used where isotope patterns (e.g. chlorine in challenge 9) could be recognized, not uniformly for all challenges.
- Enlarging the database with halogenated compounds increases candidate counts, which downstream refscore thresholding (5, or stepwise to <1000, <500, <200 molecules) must then control.

## Evidence

- [readme] README confirms pubchem subproject purpose: [evidence span withheld — anchor status: external_doc]
