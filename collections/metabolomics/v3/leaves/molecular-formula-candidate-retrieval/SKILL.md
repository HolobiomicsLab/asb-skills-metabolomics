---
name: molecular-formula-candidate-retrieval
description: Use when assigning molecular formulas to MS/MS precursor peaks from untargeted
  LC-MS data without reference spectra, and you need candidate structures from PubChem
  filtered by mass deviation.
when_to_use_negative:
- When reference spectra are available for the unknown, since matching against experimental
  spectra is more direct and reliable.
- When the compound contains elements outside C, H, N, O, P, S — the PubChem candidate
  snapshot was filtered to those elements only, so retrieval would miss valid candidates.
- When the precursor mass exceeds 1200 Da, since the candidate database was restricted
  to compounds below that mass.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MAGMa
  role: Core algorithm that fragments candidates in silico, computes substructure
    penalty scores, and ranks candidates for molecular formula assignment.
  repo: https://github.com/NLeSC/MAGMa
- name: PubChem
  role: Source of candidate molecules retrieved by precursor mass from a local snapshot
    filtered to C/H/N/O/P/S elements and mass ≤ 1200 Da.
- name: emetabolomics web interface
  role: Alternative entry point for running MAGMa via www.emetabolomics.org/magma
    instead of command line.
  repo: http://www.emetabolomics.org/results_CASMI2013.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.5702/massspectrometry.s0033
    title: Automatic Compound Annotation from Mass Spectrometry Data Using MAGMa
  source_content_hash: sha256:e3e4e8e872f0cb0874ac152b500d6316152d0bd1e134a1c1d548a275fad530b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NLeSC/MAGMa
  tools:
  - MAGMa
  - PubChem
  - emetabolomics web interface
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# molecular-formula-candidate-retrieval

## Summary

Retrieves candidate molecules matching a precursor mass from a local PubChem snapshot and ranks them by MS/MS-based substructure penalty scores, enabling molecular formula assignment for untargeted LC-MS challenge spectra.

## When to use

Use when assigning molecular formulas to MS/MS precursor peaks from untargeted LC-MS data without reference spectra, and you need candidate structures from PubChem filtered by mass deviation.

## When NOT to use

- When reference spectra are available for the unknown, since matching against experimental spectra is more direct and reliable.
- When the compound contains elements outside C, H, N, O, P, S — the PubChem candidate snapshot was filtered to those elements only, so retrieval would miss valid candidates.
- When the precursor mass exceeds 1200 Da, since the candidate database was restricted to compounds below that mass.

## Inputs

- MS/MS peak lists from the CASMI 2013 contest (converted to MAGMa hierarchical input format)
- Local PubChem database snapshot from 2 December 2013 (C/H/N/O/P/S elements only, mass ≤ 1200 Da)
- Precursor m/z and allowed mass deviation (10 ppm, absolute minimum 0.002 Da; 4 ppm for specific challenges)

## Outputs

- Ranked list of candidate molecules per challenge with penalty scores
- Top-ranked molecular formula per challenge for CASMI category 1 submission
- Per-candidate substructure annotation with penalty breakdown

## How to apply

Convert the CASMI-provided peak lists into the MAGMa hierarchical input format, where fragment peaks are nested in parentheses beneath the precursor peak. Retrieve candidate molecules from a local PubChem database snapshot (2 December 2013, compounds containing only C, H, N, O, P, and S with mass ≤ 1200 Da) within an allowed mass deviation of 10 ppm (absolute minimum 0.002 Da), tightened to 4 ppm for specific challenges. Generate substructures by applying up to 3 bond dissociations, expanding the set with single losses of hydroxyl groups (neutral loss of water) or primary amine groups (neutral loss of ammonia). Match m/z values of the MS/MS peaks against the best-scoring substructures and compute a total penalty score as an intensity-weighted average, penalizing non-matched peaks. Rank candidates by reversed penalty score, breaking ties using the PubChem refscore; when candidate lists are very large, apply a refscore threshold (e.g. 5) to reduce them to manageable sizes (<1000, <500, <200 molecules). The formula of the top-ranked candidate is the assigned molecular formula.

## Related tools

- **MAGMa** (Core algorithm that fragments candidates in silico, computes substructure penalty scores, and ranks candidates for molecular formula assignment.) — https://github.com/NLeSC/MAGMa
- **PubChem** (Source of candidate molecules retrieved by precursor mass from a local snapshot filtered to C/H/N/O/P/S elements and mass ≤ 1200 Da.)
- **emetabolomics web interface** (Alternative entry point for running MAGMa via www.emetabolomics.org/magma instead of command line.) — http://www.emetabolomics.org/results_CASMI2013.html

## Examples

```
magma --input challenge.msms --pubchem local_snapshot.db --mass-deviation 10ppm --bond-dissociations 3
```

## Evaluation signals

- Verify that all candidates retrieved fall within the stated mass deviation (10 ppm / 0.002 Da minimum, or 4 ppm where applied).
- Confirm that substructure generation did not exceed 3 bond dissociations plus single hydroxyl/primary-amine losses.
- Check that the top-ranked formula matches the CASMI ground-truth formula when available (the article reports all 8 category 1 top-ranked formulas were correct).
- Inspect that non-matched MS/MS peaks incurred the stated penalty rather than being silently dropped.
- Verify that refscore filtering was applied only when candidate sets exceeded the targeted size limits (<1000, <500, <200).

## Limitations

- A too-restrictive mass tolerance (0.002 Da) caused fragment peaks in challenge 11 to remain unexplained even though the correct tautomers were present in the candidate list.
- With only up to 3 bond dissociations, several higher-energy fragment peaks in challenge 16 (e.g. m/z 122.0408, 178.03072, 179.03857, 193.04172) could not be explained.
- Candidate retrieval is limited to the elements C, H, N, O, P, S and a maximum mass of 1200 Da in the PubChem snapshot.

## Evidence

- [methods] Substructure expansion with neutral losses: [evidence span withheld — anchor status: not_found]
- [methods] Candidate ranking by reversed penalty score: [evidence span withheld — anchor status: not_found]
- [readme] Readme repo URL for MAGMa: [evidence span withheld — anchor status: external_doc]
