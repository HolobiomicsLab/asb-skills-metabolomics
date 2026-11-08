---
name: pubchem-candidate-retrieval
description: Decision support for building candidate molecule sets from PubChem when
  annotating MS/MS or MSn spectra by in silico generated substructures, given only
  a precursor m/z (and optionally a known elemental composition) and no reference
  spectra.
when_to_use_negative:
- When reference spectra are available for library matching — this skill is for structure
  candidates retrieved by precursor mass/composition alone, not spectral library search.
- When the compound of interest contains elements outside C/H/N/O/P/S and halogens
  (e.g. metals) that neither the default nor the extended database covers.
- When a strict mass tolerance below 10 ppm is required without re-running retrieval
  — the default 10 ppm with 0.002 Da minimum caused missed demethoxycurcumin tautomers
  in challenge 11 until the tolerance was loosened.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MAGMa
  role: Retrieves candidate molecules from the local PubChem database using precursor
    mass and elemental filter; consumed as input to the substructure-annotation pipeline.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: 'Source of candidate structures: local snapshot database for mass filtering,
    plus online SDF retrieval when an elemental composition is provided.'
- name: emetabolomics web interface
  role: Alternative entry point for running MAGMa candidate retrieval and annotation
    via www.emetabolomics.org/magma.
  repo: http://www.emetabolomics.org/results_CASMI2013.html
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:9bbfbc7339dd69920a55e4973dd5d725aca1ad8adb5350624bf050e0dde588f4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# pubchem-candidate-retrieval

## Summary

Retrieve candidate molecule structures from a local PubChem database (snapshot 2 December 2013, C/H/N/O/P/S up to 1200 Da) within a given precursor mass deviation, optionally supplementing with online PubChem SDF queries or a halogen-augmented local database, to feed MAGMa candidate ranking for CASMI 2013 challenges.

## When to use

Decision support for building candidate molecule sets from PubChem when annotating MS/MS or MSn spectra by in silico generated substructures, given only a precursor m/z (and optionally a known elemental composition) and no reference spectra.

## When NOT to use

- When reference spectra are available for library matching — this skill is for structure candidates retrieved by precursor mass/composition alone, not spectral library search.
- When the compound of interest contains elements outside C/H/N/O/P/S and halogens (e.g. metals) that neither the default nor the extended database covers.
- When a strict mass tolerance below 10 ppm is required without re-running retrieval — the default 10 ppm with 0.002 Da minimum caused missed demethoxycurcumin tautomers in challenge 11 until the tolerance was loosened.

## Inputs

- Precursor m/z with ion mode assumption ([M+H]+ or [M-H]-) for each challenge
- Optional provided elemental composition (category 1 challenges) for direct online PubChem SDF retrieval
- Isotope ratio information from the MS file (e.g. chlorine isotope pattern for challenge 9)
- Local PubChem compound database snapshot (2 December 2013, C/H/N/O/P/S, up to 1200 Da)
- Halogen-augmented local PubChem database (post-deadline extension for challenges 15 and 16)

## Outputs

- Per-challenge candidate molecule lists ready for MAGMa substructure annotation and penalty-score ranking
- SDF-format candidate records from online PubChem queries
- Candidate sets for challenges 9, 15, and 16 containing the correct halogenated structures (chlorpyrifos, 2-(perfluorooctyl)ethanol, ofloxacin)

## How to apply

Convert the challenge peak lists to MAGMa input files, then retrieve candidate molecules from the local PubChem database built from the 2 December 2013 snapshot, retaining compounds of up to 1200 Da composed only of C, H, N, O, P, and S, assuming [M+H]+ or [M-H]- precursor ions. Search within a mass deviation of 10 ppm with an absolute minimum of 0.002 Da (whichever is larger for a given peak mass), tightening to 4 ppm for challenges 4, 5, and 6. When the composition is provided directly, retrieve candidates from the online PubChem service in SDF format instead of the local database. If the correct structure is absent because the default element filter excludes halogens (as for chlorpyrifos, 2-(perfluorooctyl)ethanol, and ofloxacin), either query online PubChem by mono-isotopic mass after inspecting isotope ratios, or extend the local database to include halogenated compounds. Optionally apply the PubChem refscore (threshold 5) to prune oversized candidate lists for challenges 1, 2, and 14.

## Related tools

- **MAGMa** (Retrieves candidate molecules from the local PubChem database using precursor mass and elemental filter; consumed as input to the substructure-annotation pipeline.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate structures: local snapshot database for mass filtering, plus online SDF retrieval when an elemental composition is provided.)
- **emetabolomics web interface** (Alternative entry point for running MAGMa candidate retrieval and annotation via www.emetabolomics.org/magma.) — http://www.emetabolomics.org/results_CASMI2013.html

## Examples

```
python -m pubchem.build_db --snapshot 2013-12-02 --elements C,H,N,O,P,S --max_mass 1200
```

## Evaluation signals

- Each candidate set is non-empty and the correct structure is present (e.g. chlorpyrifos in challenge 9, 2-(perfluorooctyl)ethanol in challenge 15, ofloxacin in challenge 16).
- All retrieved candidates fall within the specified mass deviation: 10 ppm with 0.002 Da absolute minimum, or 4 ppm for challenges 4, 5, and 6.
- All candidates respect the composition filter (C, H, N, O, P, S only) and the 1200 Da mass cutoff unless the halogen-augmented database is explicitly used.
- When a refscore threshold of 5 is applied, the pruned candidate list size stays below the target (<1000, <500, or <200 molecules).

## Limitations

- The initial C/H/N/O/P/S filter excludes halogen-containing molecules, causing misses for challenges 15 and 16 until a halogen-augmented database was added after the CASMI deadline.
- A mass tolerance of 10 ppm with 0.002 Da absolute minimum was too restrictive for challenge 11, where correct demethoxycurcumin tautomers were present but fragment peaks went unexplained until 0.005 Da minimum was used.
- Large candidate lists (e.g. challenges 1, 2, 14) require refscore-based filtering; without it the ranking becomes unwieldy.

## Evidence

- [methods] tightened tolerance for challenges 4-6: [evidence span withheld — anchor status: not_found]
- [discussion] halogen database extension rationale: [evidence span withheld — anchor status: not_found]
- [readme] repository and database description: [evidence span withheld — anchor status: external_doc]
