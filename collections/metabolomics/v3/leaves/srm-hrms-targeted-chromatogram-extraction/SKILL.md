---
name: srm-hrms-targeted-chromatogram-extraction
description: Use when SRM or vendor HRMS raw LC-MS files must be processed for a known
  target list of small molecules — importing precursor/fragment transitions, extracting
  ion chromatograms, integrating peaks, and quantifying via isotope dilution or external
  calibration.
when_to_use_negative:
- The goal is nontargeted feature-finding in MS1 spectra followed by run alignment
  and identification — that workflow belongs to XCMS, MZmine, or MS-DIAL, not Skyline's
  extract-known-targets paradigm.
- The task requires importing and scoring against external small-molecule fragmentation
  libraries (METLIN, MassBank, HMDB, NIST) — these workflows are stated as future
  work and are not yet implemented in Skyline.
- The input is GC-MS or ion-mobility small-molecule data, for which Skyline support
  is explicitly described as still ongoing.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Skyline
  role: 'Primary tool: molecule-centric targeted extraction, integration, and calibrated
    quantification of small-molecule SRM/HRMS data'
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: ProteoWizard (pwiz)
  role: Vendor-reader and mzML library layer through which Skyline accesses vendor
    raw formats; Skyline lives under pwiz_tools/Skyline and depends on pwiz library
    projects
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Web-based repository for sharing the Skyline documents and deposited raw data
    used in the extraction workflow
  repo: https://panoramaweb.org/
- name: METLIN
  role: Named future-work source of small-molecule fragmentation libraries for spectral
    matching (not yet integrated)
- name: MassBank
  role: Named future-work source of small-molecule fragmentation libraries for spectral
    matching (not yet integrated)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:b7f1108782362297f1071650a9f89fcddd471402cdc2407282761e512556a555
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - HMDB
  - METLIN
  - MassBank
  - Panorama
  - ProteoWizard (pwiz)
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# srm-hrms-targeted-chromatogram-extraction

## Summary

Targeted extraction of small-molecule fragment-ion chromatograms from SRM and HRMS raw data using Skyline's molecule-centric mode, covering scheduled transition lists, peak integration, and calibrated quantification across six major instrument vendors.

## When to use

Use when SRM or vendor HRMS raw LC-MS files must be processed for a known target list of small molecules — importing precursor/fragment transitions, extracting ion chromatograms, integrating peaks, and quantifying via isotope dilution or external calibration.

## When NOT to use

- The goal is nontargeted feature-finding in MS1 spectra followed by run alignment and identification — that workflow belongs to XCMS, MZmine, or MS-DIAL, not Skyline's extract-known-targets paradigm.
- The task requires importing and scoring against external small-molecule fragmentation libraries (METLIN, MassBank, HMDB, NIST) — these workflows are stated as future work and are not yet implemented in Skyline.
- The input is GC-MS or ion-mobility small-molecule data, for which Skyline support is explicitly described as still ongoing.

## Inputs

- SRM/MRM raw data files (vendor formats readable through ProteoWizard)
- HRMS raw data files (vendor formats readable through ProteoWizard)
- Transition list with precursor/fragment m/z, charge, and collision energies (imported into Skyline transition settings)
- Heavy-isotope internal standard definitions for isotope-dilution quantification
- Calibration curve / external calibration data for concentration calculation
- Measured retention times from a prior analysis for RT scheduling

## Outputs

- Extracted ion chromatograms with integrated peaks per target per transition
- Retention-time-scheduled instrument method (e.g., 1-minute window centered on measured RTs)
- Calibrated concentration table per small-molecule target
- Peak integration and transition-level QC results inspectable in Skyline
- Shared Skyline document deployable to Panorama (https://panoramaweb.org/)

## How to apply

Build a Skyline document configured for small molecules and import a transition list containing targets with precursor/fragment m/z and heavy labeled internal standards (e.g., the polyamine/arginine-pathway assay or the Biocrates p180 serum amino-acid panel). Schedule acquisition and extraction with retention-time windows centered on measured RTs — the article used a 1-minute window centered on prior measured retention times — and set product-ion m/z tolerances appropriate to the instrument (narrow for SRM, high-res for HRMS). Load raw files from the study's Panorama deposit; Skyline reads vendor formats via the ProteoWizard reader stack, so Thermo, Bruker, Waters and other vendor files can be mixed in one document. Extract fragment-ion chromatograms for each target, integrate peaks, and handle positive/negative ion-switching data within a single project as demonstrated on the purine/pyrimidine dataset. Quantify via stable-isotope dilution or external calibration, and optionally benchmark integration results against a comparable instrument-vendor software tool before export.

## Related tools

- **Skyline** (Primary tool: molecule-centric targeted extraction, integration, and calibrated quantification of small-molecule SRM/HRMS data) — https://github.com/ProteoWizard/pwiz-cpp
- **ProteoWizard (pwiz)** (Vendor-reader and mzML library layer through which Skyline accesses vendor raw formats; Skyline lives under pwiz_tools/Skyline and depends on pwiz library projects) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Web-based repository for sharing the Skyline documents and deposited raw data used in the extraction workflow) — https://panoramaweb.org/
- **METLIN** (Named future-work source of small-molecule fragmentation libraries for spectral matching (not yet integrated))
- **MassBank** (Named future-work source of small-molecule fragmentation libraries for spectral matching (not yet integrated))

## Evaluation signals

- Every target in the imported transition list yields an extracted fragment-ion chromatogram with a defensible integrated peak when manually inspected in Skyline.
- The exported method's retention-time windows match the specified schedule — a 1-minute window centered on previously measured retention times.
- SRM and HRMS acquisitions of the same targets agree on detected peaks and quantification when cross-checked against manually inspected chromatograms.
- Positive/negative-switching acquisitions (e.g., the purine/pyrimidine compounds dataset) process within a single Skyline project without splitting into separate documents.
- Concentrations reproduce the published ADNI serum amino-acid (Biocrates p180) benchmark behavior when Skyline results are compared against the instrument-vendor software.

## Limitations

- For latest-generation or vendor-specific workflows, Skyline may lag behind commercial software until support is implemented.
- Fragmentation-library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented — spectral matching against those libraries is future work.
- No single tool is best for all workflows; GC-MS and ion-mobility small-molecule support remain ongoing expansions.

## Evidence
