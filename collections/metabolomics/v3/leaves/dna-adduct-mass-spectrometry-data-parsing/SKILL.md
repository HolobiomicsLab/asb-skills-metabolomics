---
name: dna-adduct-mass-spectrometry-data-parsing
description: Use when LC-HRMS DNA adductomics data must be annotated and experimental
  MS/MS spectra need comparison against in silico predicted fragments from CFM-ID
  for DNA adducts.
when_to_use_negative:
- The input is raw unprocessed MS data without extracted fragment tables — this skill
  operates on already-extracted fragment m/z datatables
- The task is building the database itself (structure harmonization, adding 303 new
  potential adducts) rather than validating spectral agreement for annotation
- The analytes are not DNA adducts or the spectra come from a very different fragmentation
  regime where predicted/experimental overlap logic does not apply
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0654
tools:
- name: CFM-ID
  role: Generates the in silico MS/MS fragments for all 582 database adducts that
    are compared against experimental spectra
- name: MS/MS
  role: Provides the experimental tandem spectra of 15 reference standards used in
    the overlap assessment
- name: tandem HRMS/MS
  role: High-resolution fragment data source for identifying DNA adducts by spectral
    comparison
- name: DT
  role: R package (rstudio/DT) used to render the database's experimental and predicted
    fragments into interactive web datatables
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:cf0fd5f1dc190a0cb628ea7d92b49abb1229199d58d10e4807400023bb34ad1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - DT
  - MS/MS
  - tandem HRMS/MS
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# dna-adduct-mass-spectrometry-data-parsing

## Summary

A skill for parsing and interpreting DNA adduct mass spectrometry data — comparing CFM-ID-predicted MS/MS fragments against experimental tandem HRMS/MS spectra of reference standards acquired on a Vion-IMS-qTOF to support DNA adduct annotation in untargeted adductomics.

## When to use

Use when LC-HRMS DNA adductomics data must be annotated and experimental MS/MS spectra need comparison against in silico predicted fragments from CFM-ID for DNA adducts.

## When NOT to use

- The input is raw unprocessed MS data without extracted fragment tables — this skill operates on already-extracted fragment m/z datatables
- The task is building the database itself (structure harmonization, adding 303 new potential adducts) rather than validating spectral agreement for annotation
- The analytes are not DNA adducts or the spectra come from a very different fragmentation regime where predicted/experimental overlap logic does not apply

## Inputs

- Experimental MS/MS fragment datatable for 15 DNA adduct reference standards (20/40/60 eV collision energies, Vion-IMS-qTOF)
- CFM-ID predicted fragment datatable (fragments above 5% relative intensity, three energy levels)

## Outputs

- fragment_overlap_summary.csv with per-adduct overlap statistics
- Overlaid predicted-vs-experimental spectra plots (one panel per standard, 15 panels)
- Validated annotation basis for DNA adducts detected by tandem HRMS/MS

## How to apply

Load the experimental MS/MS fragment datatable for 15 reference standards (acquired at 20, 40, and 60 eV on the Vion-IMS-qTOF instrument) alongside the CFM-ID predicted fragment datatable (fragments above 5% relative intensity at three energy levels). For each of the 15 DNA adduct standards, match predicted fragment m/z values against experimental fragment m/z values at each collision energy using a mass tolerance consistent with qTOF high-resolution accuracy, paying particular attention to the characteristic [M+H-dR]+ deoxyribose-loss fragment. Compute per-adduct overlap statistics (matched fragments relative to total predicted and experimental fragments) and generate overlaid predicted-vs-experimental spectra plots, one panel per standard, mirroring Supplementary Figures S1–S15. The rationale: agreement between predicted and experimental spectra is what allows the 582-entry database's predicted fragments to be trusted for annotating adducts detected by LC-HRMS in biological samples.

## Related tools

- **CFM-ID** (Generates the in silico MS/MS fragments for all 582 database adducts that are compared against experimental spectra)
- **MS/MS** (Provides the experimental tandem spectra of 15 reference standards used in the overlap assessment)
- **tandem HRMS/MS** (High-resolution fragment data source for identifying DNA adducts by spectral comparison)
- **DT** (R package (rstudio/DT) used to render the database's experimental and predicted fragments into interactive web datatables) — https://github.com/rstudio/DT

## Examples

```
R: DT::datatable(fragment_dt) to render the MS/MS fragment datatables for the 15 standards (grounded in the README's datatable() example).
```

## Evaluation signals

- The overlap summary qualitatively reproduces the reported result: experimental and CFM-ID in silico spectra show considerable overlap, supporting DNA adduct annotation
- The characteristic [M+H-dR]+ deoxyribose-loss fragment appears among the matched fragments for each standard
- 15 comparison panels are produced, one per reference standard, matching Supplementary Figures S1–S15
- All 15 standards are represented in fragment_overlap_summary.csv with overlap counts for each collision energy (20, 40, 60 eV)
- Predicted fragments used are filtered above 5% relative intensity as generated by CFM-ID

## Limitations

- The overlap assessment is qualitative — considerable agreement supports but does not unequivocally confirm adduct identity, which additionally requires comparison with reference standards or NMR
- Only 15 reference standards have experimental MS/MS spectra; the remaining database entries rely on predicted fragments alone
- Predicted fragments were filtered to those above 5% relative intensity, so low-intensity predicted fragments are not part of the comparison
- Tentative identification via HRMS/MS still benefits from confirmation against standards or NMR

## Evidence

- [abstract] CFM-ID predicted fragments match experimental MS/MS with considerable overlap: "generating in silico MS/MS fragments for all the adducts"
- [abstract] Interactive web datatables host experimental and predicted fragments: "reporting both experimental and predicted fragments into interactive web datatables"
- [abstract] Experimental MS/MS library built from 15 reference standards: "a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [readme] DT package renders data tables in R: [evidence span withheld — anchor status: external_doc]
- [abstract] DNA adducts tentatively identified by tandem HRMS/MS: "and tentatively identified by tandem HRMS/MS"
