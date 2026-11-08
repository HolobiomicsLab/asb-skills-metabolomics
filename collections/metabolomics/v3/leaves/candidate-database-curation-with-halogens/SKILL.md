---
name: candidate-database-curation-with-halogens
description: Use when MS/MS isotope patterns (e.g. chlorine isotope ratios) or unexplained
  challenges indicate the correct molecule contains halogens absent from the default
  C, H, N, O, P, S candidate database.
when_to_use_negative:
- The correct candidate is already present in the initial C/H/N/O/P/S-only PubChem
  candidate set — extending the database only enlarges search space without benefit
- The mass tolerance needed is tighter or looser than the default 10 ppm / 0.002 Da
  rule (e.g. challenges 4, 5, 6 used 4 ppm); adjust the retrieval tolerance instead
  of augmenting the database
- Aim is molecular-formula-only annotation (category 1); the halogen supplement matters
  for structure-level candidate retrieval in category 2
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MAGMa
  role: Provides the local compound database and candidate retrieval step whose composition
    filter is extended to include halogenated PubChem compounds
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of the 2 December 2013 compound snapshot for the local database and
    of online SDF candidate records queried by mono-isotopic mass
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:59abb659d5c57cf1f4a4442050545a096dae78daea3fae4ad55afd45bb9577ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# candidate-database-curation-with-halogens

## Summary

Extends a local MAGMa compound database built from a PubChem snapshot by lifting the composition filter beyond C, H, N, O, P, and S to include halogenated compounds, so that molecules like chlorpyrifos, 2-(perfluorooctyl)ethanol, and ofloxacin become retrievable candidates within a 10 ppm (min 0.002 Da) mass window.

## When to use

Use when MS/MS isotope patterns (e.g. chlorine isotope ratios) or unexplained challenges indicate the correct molecule contains halogens absent from the default C, H, N, O, P, S candidate database.

## When NOT to use

- The correct candidate is already present in the initial C/H/N/O/P/S-only PubChem candidate set — extending the database only enlarges search space without benefit
- The mass tolerance needed is tighter or looser than the default 10 ppm / 0.002 Da rule (e.g. challenges 4, 5, 6 used 4 ppm); adjust the retrieval tolerance instead of augmenting the database
- Aim is molecular-formula-only annotation (category 1); the halogen supplement matters for structure-level candidate retrieval in category 2

## Inputs

- PubChem snapshot compound database (2 December 2013) filtered to C, H, N, O, P, S with 1200 Da mass cutoff
- MS/MS peak lists from CASMI 2013 challenges (e.g. challenges 9, 15, 16) with mono-isotopic peak masses
- SDF-format records from online PubChem candidate queries (e.g. for challenge 9)

## Outputs

- Halogen-augmented local MAGMa compound database including halogen-containing compounds up to 1200 Da
- Candidate lists per challenge retrieved at 10 ppm (min 0.002 Da) tolerance, e.g. written to halogen_candidates.csv
- Verified candidate sets containing the correct halogenated structures (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin)

## How to apply

Build the local MAGMa compound database from the PubChem snapshot (2 December 2013) but add halogen elements to the composition filter, which previously restricted compounds to C, H, N, O, P, and S, while keeping the 1200 Da mass cutoff. For peaks showing halogen isotope patterns (e.g. chlorine in challenge 9), optionally retrieve candidates online from PubChem by mono-isotopic mass, handling the returned SDF records as retrieval input. Query the halogen-augmented local database using the default mass tolerance of 10 ppm with an absolute minimum of 0.002 Da (whichever is larger for a given peak mass). Verify that the known correct halogenated structures (chlorpyrifos for challenge 9, 2-(perfluorooctyl)ethanol for challenge 15, ofloxacin for challenge 16) appear in the resulting candidate sets before writing them out (e.g. to halogen_candidates.csv) for downstream MAGMa substructure-annotation scoring.

## Related tools

- **MAGMa** (Provides the local compound database and candidate retrieval step whose composition filter is extended to include halogenated PubChem compounds) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of the 2 December 2013 compound snapshot for the local database and of online SDF candidate records queried by mono-isotopic mass)

## Examples

```
python -m pubchem.build_db --snapshot pubchem_2013-12-02 --elements C,H,N,O,P,S,F,Cl,Br,I --max-mass 1200 && magma retrieve --challenges 9,15,16 --tol 10ppm --tol-min 0.002Da --output halogen_candidates.csv
```

## Evaluation signals

- Each retrieved candidate set is non-empty for the target challenges (9, 15, 16)
- Chlorpyrifos is present in the challenge 9 candidate set, 2-(perfluorooctyl)ethanol in challenge 15, and ofloxacin in challenge 16
- Retrieved candidate masses fall within 10 ppm of the peak mass, with an absolute minimum of 0.002 Da
- The augmented database retains the 1200 Da mass cutoff while now returning compounds containing F, Cl, Br, or I
- Retrospective MAGMa processing of challenges 9, 15, and 16 yields satisfactory entries once halogenated candidates are included

## Limitations

- The halogen database was added only after the CASMI deadline, so it served retrospective supplementation rather than the original submissions
- Even with correct candidates present, too-restrictive mass tolerance (10 ppm / 0.002 Da) can leave fragment peaks unexplained, as seen for challenge 11's tautomers
- Default in silico fragmentation (up to 3 bond dissociations) may still be insufficient for halogenated molecules (e.g. challenge 16's higher-energy peaks), requiring parameter tuning beyond candidate supplementation

## Evidence

- [discussion] To overcome the missing halogen-containing candidates, a second MAGMa compound database containing halogenated PubChem compounds was added after the CASMI deadline: [evidence span withheld — anchor status: not_found]
- [methods] MAGMa retrieved candidate molecules from a local database consisting of all Pub Chem compounds on 2 December 2013: "MAGMa retrieved candidate molecules from a local database consisting of all Pub Chem compounds on 2 December 2013"
- [methods] The allowed mass deviation was set to 10 ppm, with an absolute minimum of 0.002 Da: "The allowed mass deviation was set to 10 ppm, with an absolute minimum of 0.002 Da"
- [methods] candidates with the provided elemental composition were retrieved directly from the online Pub Chem service in SDF format: "candidates with the provided elemental composition were retrieved directly from the online Pub Chem service in SDF format"
- [readme] job - Runs MAGMa calculation; pubchem - Processing of PubChem database, used to find mass candidates: [evidence span withheld — anchor status: external_doc]
