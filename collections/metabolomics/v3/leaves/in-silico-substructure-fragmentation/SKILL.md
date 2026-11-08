---
name: in-silico-substructure-fragmentation
description: Use when you need to rank candidate molecules for an unknown metabolite
  from accurate-mass MS/MS peak lists without reference spectra, e.g. candidate lists
  from PubChem for CASMI-style contests or untargeted LC-MS identification.
when_to_use_negative:
- When no accurate-mass MS/MS or MSn peak data are available and only a feature table
  without fragmentation is provided
- When reference spectra exist and a spectral-library match would suffice, since MAGMa
  is designed for annotating spectra in the absence of informative reference spectra
- When candidate structures contain elements outside the assumed set (C/H/N/O/P/S)
  or exceed the mass range, unless the halogen supplement option and candidate database
  are extended accordingly
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MAGMa
  role: Generates substructures by in silico bond dissociation, matches peaks, computes
    penalty scores, and ranks candidate structures; runs in command line mode or via
    the web interface
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules retrieved by elemental composition in SDF format
    from a local database snapshot (2 December 2013) using mass deviation and refscore
    filters
- name: emetabolomics web interface
  role: Web front-end (www.emetabolomics.org/magma) to submit MAGMa jobs and view
    ranked annotation results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:e9c8f950874aa85ce637a5d7aa5cc6af49b742c5e95eb2ff0972fa6c102d15e0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# in-silico-substructure-fragmentation

## Summary

Annotate MS/MS fragment peaks by matching them against substructures generated in silico from candidate chemical structures, computing intensity-weighted penalty scores to rank candidates — the core of the MAGMa (Ms Annotation based on in silico Generated Metabolites) approach.

## When to use

Use when you need to rank candidate molecules for an unknown metabolite from accurate-mass MS/MS peak lists without reference spectra, e.g. candidate lists from PubChem for CASMI-style contests or untargeted LC-MS identification.

## When NOT to use

- When no accurate-mass MS/MS or MSn peak data are available and only a feature table without fragmentation is provided
- When reference spectra exist and a spectral-library match would suffice, since MAGMa is designed for annotating spectra in the absence of informative reference spectra
- When candidate structures contain elements outside the assumed set (C/H/N/O/P/S) or exceed the mass range, unless the halogen supplement option and candidate database are extended accordingly

## Inputs

- MS/MS peak lists (precursor plus fragment m/z and intensities, e.g. CASMI 2013 challenge peak lists)
- Candidate molecular structures or elemental compositions to search against PubChem (SDF format via online PubChem service)
- Precursor ion adduct assumption ([M+H]+ or [M-H]-)
- MAGMa input files with fragment hierarchy in parentheses

## Outputs

- Ranked candidate structure list per challenge, ordered by reversed penalty score with PubChem refscore tie-breaking
- Per-candidate intensity-weighted total penalty scores
- Assignment of each observed MS/MS m/z value to its best scoring substructure
- Molecular formulas (top-ranked) for category 1-style submissions

## How to apply

Convert the provided MS/MS peak lists into MAGMa input format, in which the precursor and its fragment hierarchy are given in parentheses; optionally merge MS/MS spectra from different collision energies into composite spectra taking the highest intensity for ions present at both energies. Retrieve candidate molecules by elemental composition from a PubChem snapshot (here: 2 December 2013, C/H/N/O/P/S only, up to 1200 Da) within a mass deviation of 10 ppm with an absolute minimum of 0.002 Da (tighten to 4 ppm for specific challenges), assuming [M+H]+ or [M-H]- precursor ions, and apply a PubChem refscore threshold of 5 to cut down very large candidate sets. Systematically fragment each candidate by applying up to 3 bond dissociations and expand the substructure set with single losses of hydroxyl groups (neutral loss of water) or primary amine groups (neutral loss of ammonia). Match the m/z values of the MS/MS peaks with the best scoring substructures, add a penalty of 10 for non-matched peaks, and compute the total penalty score per candidate as an intensity-weighted average of substructure penalties. Rank candidates by reversed penalty score, breaking ties by PubChem refscore.

## Related tools

- **MAGMa** (Generates substructures by in silico bond dissociation, matches peaks, computes penalty scores, and ranks candidate structures; runs in command line mode or via the web interface) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules retrieved by elemental composition in SDF format from a local database snapshot (2 December 2013) using mass deviation and refscore filters)
- **emetabolomics web interface** (Web front-end (www.emetabolomics.org/magma) to submit MAGMa jobs and view ranked annotation results)

## Examples

```
git submodule update --init && python -m job --input challenge_2_peaks.magma --pubchem pubchem_20131202.db --mass-deviation 10ppm --adduct [M+H]+ --bond-dissociations 3
```

## Evaluation signals

- In the CASMI 2013 category 2 benchmark, the correct chemical structure should be ranked first in 6 of 12 submitted solutions and outside the top 5 in 3 — replicate this distribution when reproducing the workflow
- All peaks in the composite MS/MS spectrum should be matched or explicitly penalized at 10 points each for non-matched peaks
- Candidate sets after refscore filtering should be reduced stepwise to <1000, <500, and <200 molecules when stepwise threshold raising is applied
- Retrieved candidates should match the precursor within 10 ppm (absolute minimum 0.002 Da; 4 ppm for challenges 4, 5, and 6) and the assumed adduct [M+H]+ or [M-H]-
- Equally scoring candidates must be ordered deterministically by PubChem refscore

## Limitations

- Too-restrictive mass tolerance (10 ppm / 0.002 Da) caused initially unexplained fragment peaks for challenge 11 even though correct tautomers were present in the candidate list
- Default in silico fragmentation with up to 3 bond dissociations was insufficient for challenge 16, leaving higher-energy peaks (e.g. m/z=122.0408, 178.03072, 179.03857) unexplained
- Successful annotation requires inclusion of the relevant candidate molecules in the database — misses occurred when the correct candidate was absent or filtered out
- MAGMa was developed for MSn multistage data with n>2; on MS/MS its performance was comparable to, not necessarily better than, other methods

## Evidence

- [methods] A total penalty score for a given candidate is calculated as an intensity weighted average of the substructure penalties.: [evidence span withheld — anchor status: not_found]
