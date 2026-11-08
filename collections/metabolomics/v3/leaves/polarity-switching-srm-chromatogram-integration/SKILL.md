---
name: polarity-switching-srm-chromatogram-integration
description: Use when SRM/MRM raw files from a triple quadrupole contain transitions
  acquired with rapid positive/negative polarity switching and you need peak integration
  plus calibrated concentrations in a single, vendor-independent project.
when_to_use_negative:
- Data are untargeted feature-finding/alignment/identification workflows better served
  by XCMS, MZmine, or MS-DIAL rather than targeted extraction
- Data are GC-MS or ion mobility acquisitions, where Skyline small-molecule support
  is still ongoing and no single tool is best for all workflows
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Skyline
  role: 'Handles polarity-switching SRM data in a single project: imports transition
    lists and vendor raw files, integrates EICs for both polarities, and runs calibrated
    quantification'
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: ProteoWizard
  role: Provides the underlying vendor reader framework Skyline depends on for reading
    raw triple-quadrupole SRM data
  repo: https://github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web-based repository for sharing the resulting Skyline documents and quantitative
    data
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:9ba4ccbb2c6df52874290d5d34d9d0b93af75568025fd720b241c6322c20b7a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - ProteoWizard
  - Skyline
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# polarity-switching-srm-chromatogram-integration

## Summary

Process and integrate LC-SRM chromatograms from a single acquisition that rapidly switches between positive and negative ionization modes (e.g., 112 purine/pyrimidine transitions on a SCIEX 5500 with ~1.3 s cycle time), using Skyline to handle both polarities in one project with heavy internal-standard calibrated quantification.

## When to use

Use when SRM/MRM raw files from a triple quadrupole contain transitions acquired with rapid positive/negative polarity switching and you need peak integration plus calibrated concentrations in a single, vendor-independent project.

## When NOT to use

- Data are untargeted feature-finding/alignment/identification workflows better served by XCMS, MZmine, or MS-DIAL rather than targeted extraction
- Data are GC-MS or ion mobility acquisitions, where Skyline small-molecule support is still ongoing and no single tool is best for all workflows

## Inputs

- Vendor raw LC-SRM files acquired with positive/negative polarity switching (e.g., SCIEX 5500 triple quadrupole, 112 SRM transitions, ~1.3 s total cycle time)
- Transition list of small-molecule targets with heavy-labeled internal standards (e.g., IROA MSMLS compounds)
- Published reference values for verification (e.g., Table S1 concentrations)

## Outputs

- Integrated extracted ion chromatograms for both positive- and negative-mode transitions in a single Skyline project
- Calibrated analyte concentrations relative to heavy internal standards
- Exported quantitative results table comparable to the published reference values

## How to apply

Import the transition list (targets plus heavy-labeled internal standards, e.g., IROA MSMLS compounds) into Skyline, then import the vendor raw LC-SRM files (SCIEX 5500 in this study) into the same Skyline document. Configure transition settings so the instrument's polarity switching is handled within one project, and let Skyline pick and integrate extracted ion chromatograms for both positive and negative transitions. Verify peak integration and retention-time scheduling against the published assay (e.g., RT-scheduled methods with a 1-minute window centered on measured retention times). Finally run the calibrated quantification workflow to compute concentrations relative to the heavy internal standards and export results for comparison with reference values (e.g., Table S1).

## Related tools

- **Skyline** (Handles polarity-switching SRM data in a single project: imports transition lists and vendor raw files, integrates EICs for both polarities, and runs calibrated quantification) — https://github.com/ProteoWizard/pwiz-cpp
- **ProteoWizard** (Provides the underlying vendor reader framework Skyline depends on for reading raw triple-quadrupole SRM data) — https://github.com/ProteoWizard/pwiz
- **Panorama** (Web-based repository for sharing the resulting Skyline documents and quantitative data) — https://panoramaweb.org/

## Evaluation signals

- Peak integration and retention-time scheduling are consistent with the published assay for all transitions
- Both positive- and negative-mode transitions (112 SRM transitions in the study) are present and integrated within the single Skyline project
- Total cycle time of the method matches the expected ~1.3 s for the polarity-switching acquisition
- Exported concentrations reproduce the values reported in the reference table (Table S1) within acceptable agreement
- Quantification ratios of endogenous to heavy internal standards are computed for every target compound

## Limitations

- Skyline may lag behind commercial vendor software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; other packages may be preferable for some metabolomics analyses
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented in Skyline

## Evidence

- [results] Skyline successfully processed a custom purine/pyrimidine method on a SCIEX 5500, accommodating 112 SRM transitions under both ionization modes in a single LC-SRM-MS acquisition with a total cycle time of ~1.3 seconds.: [evidence span withheld — anchor status: not_found]
