---
name: skyline-targeted-metabolomics-document-building
description: Use when a targeted small-molecule or metabolite assay (SRM/MRM or HRMS)
  needs a vendor-independent Skyline document — e.g. a transition list plus SCIEX
  5500 raw files with rapid positive/negative polarity switching — to yield integrated
  peak areas and calibrated concentrations.
when_to_use_negative:
- The goal is non-targeted analysis (MS1 feature-finding, run alignment, unidentified
  features) — use XCMS, MZmine, or MS-DIAL instead, since Skyline's approach extracts
  known targets
- The data are GC-MS or ion mobility small-molecule acquisitions, where Skyline support
  is still in-progress
- The input is already a processed feature/quantification table rather than raw vendor
  data and a transition list
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Skyline
  role: 'Primary tool for building the targeted document: importing transition lists
    and vendor raw files, picking/ integrating peaks for both polarities, RT-scheduled
    method export, and calibrated quantification'
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Web-based repository for sharing the resulting Skyline documents and targeted
    methods
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:2f4f6f9d154cc11906d7f7ce8d006b35ea03606a323eb748c5ae71dce204919d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# skyline-targeted-metabolomics-document-building

## Summary

Build a single Skyline document for targeted small-molecule quantification: import an SRM transition list (with heavy-labeled internal standards, e.g. IROA MSMLS) plus vendor raw LC-SRM files, integrate extracted ion chromatograms, and export calibrated concentrations. Skyline is molecule-centric and vendor-agnostic, supporting data from six major MS instrument vendors in one project.

## When to use

Use when a targeted small-molecule or metabolite assay (SRM/MRM or HRMS) needs a vendor-independent Skyline document — e.g. a transition list plus SCIEX 5500 raw files with rapid positive/negative polarity switching — to yield integrated peak areas and calibrated concentrations.

## When NOT to use

- The goal is non-targeted analysis (MS1 feature-finding, run alignment, unidentified features) — use XCMS, MZmine, or MS-DIAL instead, since Skyline's approach extracts known targets
- The data are GC-MS or ion mobility small-molecule acquisitions, where Skyline support is still in-progress
- The input is already a processed feature/quantification table rather than raw vendor data and a transition list

## Inputs

- SRM/MRM transition list (precursor/product m/z, collision energies) including heavy-labeled internal standards, e.g. IROA MSMLS compounds
- Vendor raw LC-SRM data files (e.g. SCIEX 5500 with rapid positive/negative polarity switching)
- Measured retention times from a prior analysis (for RT scheduling)
- Calibration/quantification settings (stable-isotope dilution or external calibration)

## Outputs

- Skyline document (.sky) containing transitions and imported raw data for both polarities in one project
- Retention-time-scheduled instrument method export (1-minute window centered on measured RTs)
- Integrated peak areas per transition and reviewed peak boundaries
- Concentration table for target compounds relative to heavy internal standards (comparable to published Table S1)

## How to apply

Import the transition list into Skyline, including light analytes and their heavy-labeled internal standards (e.g. from IROA MSMLS), then import the vendor raw LC-SRM files (e.g. SCIEX 5500 .wiff acquired with positive/negative polarity switching) into the same document. Configure transition settings so polarity switching is handled within a single project and peaks are picked for both positive and negative transitions; collision energy prediction/optimization is set under Settings->Transition Settings->Prediction. Export a retention-time-scheduled method using a 1-minute window centered on measured retention times. Integrate the extracted ion chromatograms for each transition, manually reviewing peak integration boundaries, then run the calibrated quantification workflow (stable-isotope dilution relative to heavy standards) to compute concentrations. Export the quantitative results and diff them against reference values (e.g. published Table S1) to verify the reproduction.

## Related tools

- **Skyline** (Primary tool for building the targeted document: importing transition lists and vendor raw files, picking/ integrating peaks for both polarities, RT-scheduled method export, and calibrated quantification) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Web-based repository for sharing the resulting Skyline documents and targeted methods) — https://panoramaweb.org/

## Evaluation signals

- All transitions (e.g. 112 purine/pyrimidine SRM transitions) appear and are integrated in one Skyline document covering both positive and negative ionization modes
- Total cycle time is consistent with the published acquisition (~1.3 seconds) despite polarity switching
- Exported method uses 1-minute retention-time windows centered on measured RTs
- Calibrated concentrations reproduce the reference values (e.g. Table S1) within reasonable agreement
- Peak integration for both polarities completed without needing to split the data into separate projects

## Limitations

- Skyline may lag behind commercial vendor software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; GC-MS and ion mobility small-molecule support are still expanding
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented

## Evidence
