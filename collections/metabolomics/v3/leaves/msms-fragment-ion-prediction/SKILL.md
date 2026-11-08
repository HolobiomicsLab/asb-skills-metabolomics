---
name: msms-fragment-ion-prediction
description: Use when public/commercial MS/MS libraries lack spectra for a compound
  class (e.g., <50 acylcarnitine structures) and MSI level 2 annotation requires predicted
  fragment ions.
when_to_use_negative:
- Compound class is already covered by experimental spectra and authentic standards
  are available for MSI level 1 identification (precursor mass, retention time, and
  MS/MS matching)
- Input is a single unknown spectrum without any template class or formula constraint
  — use de novo fragmentation-tree tools rather than a template-derived library
- Goal is confirmation of already-annotated metabolites rather than annotation of
  the previously unannotated spectral space
tools:
- name: CarniBlast
  role: In silico MS/MS library specifically geared at acylcarnitines; template-rule
    fragment prediction produced 2,400 entries
- name: SIRIUS/CSI:FingerID
  role: In silico fragmentation tool for MSI level 3 annotation of spectra not covered
    by template libraries; benchmarked against MSI level 1 compounds
  repo: https://github.com/sirius-ms/sirius
- name: NIST17 hybrid search
  role: Hybrid spectral search against in silico annotations to assign MSI level 3
    compounds and their ClassyFire classes
- name: RDKit
  role: SMILES generation for combinatorial enumeration of acyl chain lengths and
    modification combinations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:87cc2a7e0bcc876bf12c850efd28b3b51274c01cbeda7fe330b3940ca78d9a73
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CarniBlast
  - NIST17
  - NIST17 hybrid search
  - RDKit
  - SIRIUS/CSI:FingerID
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# msms-fragment-ion-prediction

## Summary

Predict tandem MS/MS fragment ions from chemical structure templates to build spectral libraries (CarniBlast-style for acylcarnitines, LipidBlast-style rules) or run in silico fragmentation (CSI:FingerID), enabling structure annotation where experimental libraries are sparse.

## When to use

Use when public/commercial MS/MS libraries lack spectra for a compound class (e.g., <50 acylcarnitine structures) and MSI level 2 annotation requires predicted fragment ions.

## When NOT to use

- Compound class is already covered by experimental spectra and authentic standards are available for MSI level 1 identification (precursor mass, retention time, and MS/MS matching)
- Input is a single unknown spectrum without any template class or formula constraint — use de novo fragmentation-tree tools rather than a template-derived library
- Goal is confirmation of already-annotated metabolites rather than annotation of the previously unannotated spectral space

## Inputs

- Carnitine core structure template (SMILES) with variable acyl chain attachment point
- Acyl chain enumeration parameters: chain lengths and six modification classes (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized)
- Precursor accurate mass and DDA MS/MS spectra from HILIC-Q Exactive HF and CSH-Q Exactive HF assays
- Molecular formulas from isotope pattern analysis (e.g., via SIRIUS)

## Outputs

- NIST MSP-format spectral library file (carniblast_library.msp) with 2,400 acylcarnitine entries
- Per-entry structure metadata: name, formula, SMILES, ion mode
- Predicted fragment peak lists per candidate structure
- MSI level 2/3 structure annotations assigned against the predicted library

## How to apply

Define a core structure template (carnitine headgroup plus variable acyl chain attachment) and combinatorially enumerate acyl chain lengths and modification classes — saturated, unsaturated, hydroxyl, keto, dicarboxylic, and oxidized — via RDKit SMILES generation until the candidate set is complete (2,400 acylcarnitine species for CarniBlast). Predict tandem MS/MS fragments for each candidate from template rules (carnitine headgroup fragment ions plus acyl-chain-diagnostic losses), following the LipidBlast in silico style. Assemble precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode) into a NIST MSP-format library file. For non-template chemistry, run SIRIUS/CSI:FingerID in silico fragmentation and the NIST17 hybrid search to annotate remaining spectra at MSI level 3, then benchmark top-ranked predictions against MSI level 1 identifications (which gave 40% accuracy in this study).

## Related tools

- **CarniBlast** (In silico MS/MS library specifically geared at acylcarnitines; template-rule fragment prediction produced 2,400 entries)
- **SIRIUS/CSI:FingerID** (In silico fragmentation tool for MSI level 3 annotation of spectra not covered by template libraries; benchmarked against MSI level 1 compounds) — https://github.com/sirius-ms/sirius
- **NIST17 hybrid search** (Hybrid spectral search against in silico annotations to assign MSI level 3 compounds and their ClassyFire classes)
- **RDKit** (SMILES generation for combinatorial enumeration of acyl chain lengths and modification combinations)

## Evaluation signals

- Parse carniblast_library.msp and verify the total number of spectral entries equals 2,400
- Verify all six acyl-chain modification classes (saturated, unsaturated, hydroxyl, keto, dicarboxylic, oxidized) are represented in the library
- Each MSP entry must carry precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode)
- Benchmark top-ranked CSI:FingerID annotations against MSI level 1 identifications; expected accuracy on the order of the reported 40%
- CarniBlast-derived annotations should contribute to the 578 MSI level 2 assignments (precursor accurate mass plus MS/MS matching)

## Limitations

- CSI:FingerID top-ranked annotations yielded only 40% accuracy against MSI level 1 identified compounds; in silico structure assignments require validation with authentic chemicals
- The field lacks generally accepted, validated automated calculations of identification confidence levels with false-discovery rate assessments in metabolomics MS/MS
- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are for academic research and education use only; commercial use requires a Bright Giant license
- MSI level 3 chemical classifications may differ between subjects due to diet and chemical exposure, complicating interpretation in epidemiology studies

## Evidence

- [other] Predict the tandem MS/MS fragments for each candidate structure from the template rules (carnitine headgroup fragment ions plus acyl-chain-diagnostic losses), in the style of the LipidBlast: [evidence span withheld — anchor status: not_found]
- [other] Assemble precursor m/z, predicted fragment peaks, and structure metadata (name, formula, SMILES, ion mode) for every candidate into a NIST MSP-format library file.: [evidence span withheld — anchor status: not_found]
