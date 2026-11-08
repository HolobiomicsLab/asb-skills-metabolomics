---
name: precursor-mass-defect-computation
description: Use when MS/MS spectra of suspected chemically modified metabolites (methylation,
  acetylation, phosphorylation, hydroxylation) must be associated with library spectra
  of their non-modified parent compounds by measuring precursor and fragment mass
  shifts.
when_to_use_negative:
- The spectrum already matches a library entry directly at MSI level 2 by precursor
  accurate mass and MS/MS matching — no hybrid head-to-tail comparison is needed
- The input is MS1-only data with no MS/MS fragmentation, since fragment mass shifts
  cannot be computed
- No plausible non-modified parent compound exists in the spectral library, leaving
  no parent spectrum to compare head-to-tail
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NIST hybrid search
  role: Head-to-tail search mechanism that associates modified-metabolite spectra
    with best-scoring related parent library spectra in NIST17
- name: NIST17
  role: Spectral library whose non-modified parent compound entries serve as the reference
    side of the head-to-tail comparison
- name: SIRIUS
  role: Java-based LC-MS/MS analysis framework; its demo-data package supplies the
    modified/parent MS/MS spectral pairs used for the comparison
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:1dc2a7e00e8438b5378febda8e458753974077603a71414108d386af5f64448b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - NIST hybrid search
  - NIST17
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# precursor-mass-defect-computation

## Summary

Computes precursor mass shifts (and consistent fragment mass shifts) between experimental modified-metabolite MS/MS spectra and their non-modified parent library entries, so that modified compounds (e.g., methyladenosine vs adenosine) can be linked to library spectra via the NIST17 hybrid search head-to-tail comparison mechanism.

## When to use

Use when MS/MS spectra of suspected chemically modified metabolites (methylation, acetylation, phosphorylation, hydroxylation) must be associated with library spectra of their non-modified parent compounds by measuring precursor and fragment mass shifts.

## When NOT to use

- The spectrum already matches a library entry directly at MSI level 2 by precursor accurate mass and MS/MS matching — no hybrid head-to-tail comparison is needed
- The input is MS1-only data with no MS/MS fragmentation, since fragment mass shifts cannot be computed
- No plausible non-modified parent compound exists in the spectral library, leaving no parent spectrum to compare head-to-tail

## Inputs

- Experimental MS/MS spectra of modified metabolites (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine)
- NIST17 mass spectral library entries of non-modified parent compounds (adenosine, threonine, arginine, methionine)
- Demo-data package from github.com/sirius-ms/sirius containing the four documented modified/parent spectral pairs

## Outputs

- Per-pair precursor mass shift values (e.g., +14 Da methyl, +42 Da acetyl, +80 Da phospho, +16 Da hydroxy)
- Shifted fragment peak correspondences between modified spectrum and parent library spectrum
- hybrid_search_verification.csv with columns: modified metabolite, precursor shift, fragment shifts, best parent hit, match score

## How to apply

Load the modified-metabolite MS/MS spectra (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine) and their parent library entries (adenosine, threonine, arginine, methionine) from the sirius-ms demo-data package. For each modified spectrum, compute the precursor mass shift relative to its parent compound's library spectrum, then identify candidate fragment mass shifts consistent with the known modification: +14 Da for methylation, +80 Da for phosphorylation, +16 Da for hydroxylation, and +42 Da for acetylation. Implement the head-to-tail spectral comparison used by the NIST hybrid search, aligning modified-spectrum peaks to parent-library peaks using these precursor and fragment shifts, and score each match by the number and intensity of shifted peak correspondences. Record the best-scoring parent library entry and its score per modified spectrum in a results table, then verify that each modified spectrum associates with its biologically expected parent as the top-scoring match. This follows the article's use of the NIST hybrid search as the mechanism for annotating MSI level 3 compounds that lack direct library matches.

## Related tools

- **NIST hybrid search** (Head-to-tail search mechanism that associates modified-metabolite spectra with best-scoring related parent library spectra in NIST17)
- **NIST17** (Spectral library whose non-modified parent compound entries serve as the reference side of the head-to-tail comparison)
- **SIRIUS** (Java-based LC-MS/MS analysis framework; its demo-data package supplies the modified/parent MS/MS spectral pairs used for the comparison) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Each of the four modified spectra (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine) returns its biologically expected parent (adenosine, threonine, arginine, methionine) as the top-scoring hybrid-search hit
- Computed precursor shifts match the known modification masses: ~+14 Da (methylation), ~+80 Da (phosphorylation), ~+16 Da (hydroxylation), ~+42 Da (acetylation)
- Each pair yields a non-empty set of fragment peaks aligned at the modification-consistent fragment mass shift, scored by number and intensity of shifted correspondences
- hybrid_search_verification.csv contains all five required columns (modified metabolite, precursor shift, fragment shifts, best parent hit, match score) and exactly four data rows with expected parent hits

## Limitations

- NIST17 hybrid search structure assignments still require validation with authentic chemicals; many assignments are only 'worthy to be validated by acquiring spectra from corresponding authentic chemicals'
- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals, so hybrid-search links classify rather than confirm structures (MSI level 3, not level 1 or 2)
- Only documented modification types (methylation, phosphorylation, hydroxylation, acetylation) with known mass shifts can be handled; unknown modification masses have no shift to search for

## Evidence

- [readme] SIRIUS demo-data source for spectral pairs: [evidence span withheld — anchor status: external_doc]
