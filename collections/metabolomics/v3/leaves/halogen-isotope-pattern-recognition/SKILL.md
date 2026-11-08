---
name: halogen-isotope-pattern-recognition
description: Use when MS/MS isotope ratios suggest halogens (e.g. chlorine M+2 patterns)
  or when default MAGMa candidate retrieval limited to C,H,N,O,P,S returns no plausible
  candidates for a challenge peak mass.
when_to_use_negative:
- When the correct candidate is already present in the initial C,H,N,O,P,S-restricted
  PubChem candidate set — the article notes some missed identifications (e.g. challenge
  11) were due to over-restrictive mass tolerance, not missing halogen candidates
- When candidate lists are excessively large and the problem is ranking/filtering
  (use refscore thresholds of 5 or stepwise <1000/<500/<200 reductions instead)
- When unexplained fragment peaks stem from insufficient in silico fragmentation depth
  (e.g. challenge 16's higher-energy peaks needing more than 3 bond dissociations)
  rather than missing candidates
tools:
- name: MAGMa
  role: Annotates MS/MS peaks against candidate molecules from the halogen-augmented
    local compound database via in silico substructure matching and penalty-score
    ranking
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: 'Source of candidate molecules: the 2 December 2013 local snapshot (halogen-filtered)
    plus online SDF retrieval by mono-isotopic mass for challenge 9'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:f375b2618aa521322d601830a534490654a943e7ce94e37620e41e3d2010ebf1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# halogen-isotope-pattern-recognition

## Summary

A candidate-retrieval extension for MAGMa that recognizes halogen atoms (e.g. chlorine from MS isotope ratios) and augments the local PubChem compound database (2 December 2013 snapshot, mass cutoff 1200 Da) — originally restricted to C, H, N, O, P, and S — with halogenated compounds so challenges like CASMI 9, 15, and 16 can be annotated.

## When to use

Use when MS/MS isotope ratios suggest halogens (e.g. chlorine M+2 patterns) or when default MAGMa candidate retrieval limited to C,H,N,O,P,S returns no plausible candidates for a challenge peak mass.

## When NOT to use

- When the correct candidate is already present in the initial C,H,N,O,P,S-restricted PubChem candidate set — the article notes some missed identifications (e.g. challenge 11) were due to over-restrictive mass tolerance, not missing halogen candidates
- When candidate lists are excessively large and the problem is ranking/filtering (use refscore thresholds of 5 or stepwise <1000/<500/<200 reductions instead)
- When unexplained fragment peaks stem from insufficient in silico fragmentation depth (e.g. challenge 16's higher-energy peaks needing more than 3 bond dissociations) rather than missing candidates

## Inputs

- CASMI 2013 MS/MS peak lists converted to MAGMa input files
- Local MAGMa compound database built from the PubChem snapshot of 2 December 2013
- SDF records retrieved from the online PubChem service
- Peak mass lists with mono-isotopic masses and isotope ratio information

## Outputs

- halogen_candidates.csv with candidate lists for challenges 9, 15, and 16
- Non-empty candidate sets containing the correct halogenated structures (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin)

## How to apply

First inspect the MS input file for isotope ratio signatures indicative of halogens, such as chlorine atom patterns, and for challenges like CASMI 9 retrieve candidates via an online PubChem query on the mono-isotopic mass, handling records returned in SDF format. For challenges 15 and 16, extend the local MAGMa compound database built from the 2 December 2013 PubChem snapshot by adding halogen elements to the composition filter, keeping the 1200 Da mass cutoff. Retrieve candidates from the halogen-augmented database using the default mass deviation of 10 ppm with an absolute minimum of 0.002 Da (whichever is larger for a given peak mass). Write the resulting candidate lists for challenges 9, 15, and 16 to halogen_candidates.csv, then verify each set is non-empty and contains the expected halogenated structure: chlorpyrifos (challenge 9), 2-(perfluorooctyl)ethanol (challenge 15), and ofloxacin (challenge 16). This extension is what allowed satisfactory retrospective MAGMa processing of challenges 9, 15, and 16 after the original C,H,N,O,P,S-only filter excluded the correct molecules.

## Related tools

- **MAGMa** (Annotates MS/MS peaks against candidate molecules from the halogen-augmented local compound database via in silico substructure matching and penalty-score ranking) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules: the 2 December 2013 local snapshot (halogen-filtered) plus online SDF retrieval by mono-isotopic mass for challenge 9)

## Evaluation signals

- Each retrieved candidate set for challenges 9, 15, and 16 is non-empty
- The correct halogenated structure is present: chlorpyrifos (9), 2-(perfluorooctyl)ethanol (15), ofloxacin (16)
- Mass retrieval used 10 ppm with an absolute minimum of 0.002 Da (whichever is larger per peak mass)
- The extended database still enforces the 1200 Da mass cutoff from the original PubChem snapshot build
- Satisfactory MAGMa entries are generated retrospectively for challenges 9, 15, and 16

## Limitations

- The halogen-augmented database was added only after the CASMI deadline as a retrospective fix, so it was not part of the original contest submissions
- The 10 ppm / 0.002 Da mass deviation can be too restrictive for proper MS/MS interpretation, as seen for challenge 11 where correct tautomers were already in the candidate list
- For challenge 16, even with correct candidates, several higher-energy peaks (m/z 122.0408–221.07309) remained unexplained with up to 3 bond dissociations, indicating insufficient fragmentation depth is a separate failure mode
- Whether the extended MAGMa interface with the halogen supplement option was versioned or released is not documented

## Evidence

- [readme] pubchem subproject purpose: [evidence span withheld — anchor status: external_doc]
