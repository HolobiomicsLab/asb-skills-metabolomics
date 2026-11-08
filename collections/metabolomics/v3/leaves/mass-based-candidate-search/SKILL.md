---
name: mass-based-candidate-search
description: Use when a CASMI-style MS/MS challenge provides an elemental composition
  or accurate mono-isotopic mass and you need candidate chemical structures for annotation;
  also when initial candidates miss the correct halogenated structure.
when_to_use_negative:
- When reference MS/MS spectra for the query molecule are available and library-spectrum
  matching can be used instead of candidate retrieval from PubChem.
- When the goal is formula-only annotation without any structural candidates, as for
  the CASMI category 1 molecular-formula contest.
- When the mass of the unknown exceeds the 1200 Da database cutoff and the local PubChem
  snapshot must not be extended.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MAGMa
  role: Provides the mass-based candidate-retrieval step from the local PubChem-derived
    compound database prior to substructure fragmentation and penalty-score ranking.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules, queried online in SDF format or snapshotted
    locally (2 December 2013) into the MAGMa compound database.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:78728ce80fed5d54a60b9f2cbeda7ec0dc75ec85eb4474e6c135445fa3ed8cf4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass-based-candidate-search

## Summary

Search chemical databases (local PubChem snapshots or the online PubChem SDF service) for candidate molecules matching an observed accurate mass within a defined ppm tolerance, as a prerequisite for MAGMa substructure-based MS/MS annotation. Includes extending the search space with halogen-augmented candidates when a query's correct structure falls outside the initial CHNOPS-composition-filtered database.

## When to use

Use when a CASMI-style MS/MS challenge provides an elemental composition or accurate mono-isotopic mass and you need candidate chemical structures for annotation; also when initial candidates miss the correct halogenated structure.

## When NOT to use

- When reference MS/MS spectra for the query molecule are available and library-spectrum matching can be used instead of candidate retrieval from PubChem.
- When the goal is formula-only annotation without any structural candidates, as for the CASMI category 1 molecular-formula contest.
- When the mass of the unknown exceeds the 1200 Da database cutoff and the local PubChem snapshot must not be extended.

## Inputs

- CASMI 2013 MS/MS peak lists converted to MAGMa input format
- Accurate mono-isotopic mass or provided elemental composition per challenge
- Local MAGMa compound database of all PubChem compounds on 2 December 2013 (CHNOPS-composition-filtered, ≤1200 Da)
- Online PubChem SDF-format records retrieved by mass query
- Halogen-augmented MAGMa compound database (halogens added to the composition filter)

## Outputs

- Candidate molecule lists per challenge within the mass tolerance (e.g. <1000, <500, <200 after refscore filtering)
- Halogen-supplemented candidate lists for challenges 9, 15, and 16 (halogen_candidates.csv)
- Candidate sets feeding MAGMa substructure matching and penalty-score ranking

## How to apply

Retrieve candidates from the local MAGMa compound database built from the PubChem snapshot of 2 December 2013, using the observed accurate mass (or the provided elemental composition) with a mass deviation of 10 ppm and an absolute minimum of 0.002 Da; the composition filter restricted the default database to elements C, H, N, O, P, and S, keeping compounds up to a mass cutoff of 1200 Da. Tighten the tolerance to 4 ppm for specific challenges (4, 5, 6) where a narrower window is justified. When the correct structure contains elements outside the CHNOPS filter — e.g. chlorine confirmed by isotope ratios in the MS file, or fluorine in challenges 15 and 16 — extend the database by adding halogen elements to the composition filter, optionally query the online PubChem service in SDF format, and re-run retrieval; write the supplemented lists (e.g. halogen_candidates.csv) covering challenges 9, 15, and 16. Validate retrieval by confirming the correct structure (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin) is present in the candidate set before downstream MAGMa substructure matching and ranking by reversed penalty score.

## Related tools

- **MAGMa** (Provides the mass-based candidate-retrieval step from the local PubChem-derived compound database prior to substructure fragmentation and penalty-score ranking.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules, queried online in SDF format or snapshotted locally (2 December 2013) into the MAGMa compound database.)

## Examples

```
MAGMa retrieve candidates --mass <mono-isotopic mass> --tolerance 10ppm --min 0.002Da --db pubchem_2013-12-02 [--halogen-db] > halogen_candidates.csv
```

## Evaluation signals

- Each candidate set is non-empty and contains the correct challenge structure, consistent with retrospective MAGMa processing of challenges 9, 15, and 16.
- Retrieved candidates fall within the applied mass deviation window (10 ppm, absolute minimum 0.002 Da; 4 ppm for challenges 4, 5, 6).
- Candidate lists reduce to <1000, <500, and <200 molecules under stepwise refscore thresholding (threshold value 5).
- Halogenated challenges yield the correct structure (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin) only after the database composition filter is extended with halogens.
- Downstream MAGMa penalty-score ranking places the correct candidate among the top-ranked structures when retrieval was correct.

## Limitations

- The default composition filter (C, H, N, O, P, S only) omitted halogenated compounds, causing misses for challenges 9, 15, and 16 until the database was augmented after the CASMI deadline.
- Too-restrictive mass deviation (e.g. 0.002 Da combined with the ppm window) left challenge 11 peaks unexplained even though the correct tautomers were already in the candidate list.
- With only up to 3 bond dissociations, several higher-energy peaks for challenge 16 (m/z=122.0408, 178.03072, 179.03857, 193.04172, 194.02574, 199.05114, 205.04175, 219.05744, 221.07309) remained unannotated.
- The halogen-augmented database was added after the CASMI deadline, so its candidates were not part of the submitted solutions.

## Evidence

- [methods] narrower tolerance: [evidence span withheld — anchor status: not_found]
- [other] halogen supplement source card: [evidence span withheld — anchor status: not_found]
- [readme] repo licensing: [evidence span withheld — anchor status: external_doc]
