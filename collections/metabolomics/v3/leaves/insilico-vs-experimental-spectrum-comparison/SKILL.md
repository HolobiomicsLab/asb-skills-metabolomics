---
name: insilico-vs-experimental-spectrum-comparison
description: Use when a spectral library candidate has CFM-ID (or similar) in silico
  fragments and a matching experimental tandem HRMS/MS spectrum, and you must quantify/visualize
  fragment overlap to validate predicted fragments before using them for compound
  annotation.
when_to_use_negative:
- No experimental MS/MS spectrum exists for the compound of interest (nothing to compare
  against predicted fragments)
- Spectra come from low-resolution single-stage MS rather than tandem HRMS/MS, so
  fragment-level matching is not meaningful
- Predicted fragments were not thresholded (e.g. the >5% relative-intensity filter
  was not applied), which would inflate apparent overlap with noise peaks
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: CFM-ID
  role: Generates the in silico MS/MS fragments for each adduct that are matched against
    the experimental spectra
- name: MS/MS
  role: Experimental tandem HRMS/MS acquisition of the 15 reference standards (Vion
    IMS-qTOF at 20/40/60 eV) supplying the measured fragment spectra
- name: tandem HRMS/MS
  role: High-resolution tandem MS/MS mode providing accurate fragment m/z values for
    overlap matching with qTOF-level tolerance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:365dacc48cec2c9abd629d19103f7442e896a1709a33d128d7142c1eef65677b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MS/MS
  - tandem HRMS/MS
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# insilico-vs-experimental-spectrum-comparison

## Summary

Per-compound overlap assessment between CFM-ID–predicted in silico MS/MS fragments and experimentally acquired tandem HRMS/MS spectra (here: 15 DNA adduct reference standards on a Vion IMS-qTOF at 20/40/60 eV), used to justify predicted fragments for annotation.

## When to use

Use when a spectral library candidate has CFM-ID (or similar) in silico fragments and a matching experimental tandem HRMS/MS spectrum, and you must quantify/visualize fragment overlap to validate predicted fragments before using them for compound annotation.

## When NOT to use

- No experimental MS/MS spectrum exists for the compound of interest (nothing to compare against predicted fragments)
- Spectra come from low-resolution single-stage MS rather than tandem HRMS/MS, so fragment-level matching is not meaningful
- Predicted fragments were not thresholded (e.g. the >5% relative-intensity filter was not applied), which would inflate apparent overlap with noise peaks

## Inputs

- Experimental MS/MS fragment table for reference standards (m/z, intensity, collision energy 20/40/60 eV, qTOF)
- CFM-ID predicted fragment table (>5% relative intensity, three energy levels)
- Standard/adduct identifiers and molecular formulas (e.g. from .mol/.cdxml/SMILES entries in the database)

## Outputs

- Per-adduct overlap statistics table (matched vs total predicted and experimental fragments, e.g. fragment_overlap_summary.csv)
- Overlaid predicted-vs-experimental spectrum plots, one panel per standard
- Assessment of which standards show sufficient overlap to validate predicted fragments for annotation

## How to apply

Load the experimental fragment table (each reference standard acquired at collision energies of 20, 40, and 60 eV) and the CFM-ID predicted fragment table (predicted fragments filtered to >5% relative intensity, generated at three energy levels). For each standard, match predicted fragment m/z values against experimental fragment m/z values within a mass tolerance consistent with qTOF high-resolution accuracy, per collision energy. Track overlap counts per adduct — matched fragments relative to total predicted and total experimental fragments — giving special weight to the characteristic [M+H-dR]+ deoxyribose-loss fragment for DNA adducts. Produce overlaid predicted-vs-experimental spectra plots (e.g. one panel per standard, as in the article's Supplementary Figures S1–S15) and write overlap statistics to a summary file (e.g. fragment_overlap_summary.csv). Considerable overlap validates using the CFM-ID predicted fragments to support annotation of adducts lacking reference standards.

## Related tools

- **CFM-ID** (Generates the in silico MS/MS fragments for each adduct that are matched against the experimental spectra)
- **MS/MS** (Experimental tandem HRMS/MS acquisition of the 15 reference standards (Vion IMS-qTOF at 20/40/60 eV) supplying the measured fragment spectra)
- **tandem HRMS/MS** (High-resolution tandem MS/MS mode providing accurate fragment m/z values for overlap matching with qTOF-level tolerance)

## Evaluation signals

- The overlap summary table contains one row per standard (15 rows) with matched and total fragment counts for both predicted and experimental sides
- Per-adduct overlap includes the characteristic [M+H-dR]+ deoxyribose-loss fragment for DNA adducts
- Only predicted fragments above the 5% relative-intensity threshold are counted in the comparison
- Overlay plots show m/z-matched peaks at each collision energy (20, 40, 60 eV) within the stated qTOF mass tolerance
- The qualitative conclusion — considerable overlap between experimental and CFM-ID predicted spectra — is reproduced from the matched-fragment statistics

## Limitations

- The experimental library is preliminary: only 15 reference standards were acquired, so overlap validation covers a small subset of the 582-entry database
- Overlap is a qualitative validation ('considerable overlap') rather than a formal spectral-similarity score or cutoff, so borderline compounds may still need reference-standard confirmation or NMR
- Mass tolerance depends on the instrument class (qTOF here); applying the same matching on instruments with different accuracy would need re-tuned tolerances

## Evidence

- [abstract] considerable overlap validates predicted fragments: "generating in silico MS/MS fragments for all the adducts"
- [abstract] preliminary spectral library from 15 reference standards: "a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] both experimental and predicted fragments reported: "reporting both experimental and predicted fragments into interactive web datatables"
- [other] task card: compare CFM-ID predictions with experimental MS/MS of 15 standards: [evidence span withheld — anchor status: not_found]
