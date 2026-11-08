---
name: skyline-small-molecule-data-processing
description: Use when raw LC-SRM/LC-MS vendor files for targeted small-molecule assays
  (e.g., arginine/polyamine panel with heavy-labeled internal standards) must be imported,
  peak-picked, RT-scheduled, and calibrated into concentrations.
when_to_use_negative:
- The analysis goal is untargeted feature-finding and alignment of unknowns — the
  article points to XCMS, MZmine, and MS-DIAL for nontargeted workflows, while Skyline
  extracts known targets
- Data are GC-MS or ion-mobility small-molecule acquisitions — support for these is
  still ongoing in Skyline
- The workflow depends on small-molecule fragmentation library searches (METLIN, MassBank,
  HMDB, NIST) — these workflows are not yet implemented in Skyline
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: 'Primary tool: imports transition lists, performs RT-scheduled small-molecule
    peak picking and integration, calibrated quantification, and method export'
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: ProteoWizard/pwiz
  role: Underlying open-source libraries that enable direct reading of many vendor
    raw data formats into Skyline
  repo: https://github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web-based repository for sharing Skyline documents and targeted methods/data
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:5245e6b927c20e165f338f625c55bea33f7261367991c9a76573eb028183713b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - ProteoWizard/pwiz
  - Skyline
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# skyline-small-molecule-data-processing

## Summary

Process targeted LC-SRM/HRMS small-molecule quantification data in Skyline: import vendor raw files and a transition list, integrate chromatographic peaks with heavy-labeled internal standards, and export calibrated analyte concentrations. Skyline is a vendor-independent, molecule-centric tool spanning six major MS instrument vendors.

## When to use

Use when raw LC-SRM/LC-MS vendor files for targeted small-molecule assays (e.g., arginine/polyamine panel with heavy-labeled internal standards) must be imported, peak-picked, RT-scheduled, and calibrated into concentrations.

## When NOT to use

- The analysis goal is untargeted feature-finding and alignment of unknowns — the article points to XCMS, MZmine, and MS-DIAL for nontargeted workflows, while Skyline extracts known targets
- Data are GC-MS or ion-mobility small-molecule acquisitions — support for these is still ongoing in Skyline
- The workflow depends on small-molecule fragmentation library searches (METLIN, MassBank, HMDB, NIST) — these workflows are not yet implemented in Skyline

## Inputs

- Vendor raw LC-SRM/LC-MS data files (multiple instrument vendors, read via ProteoWizard/pwiz)
- Small-molecule transition list with precursor/product m/z, retention times, and heavy-labeled internal standards
- Calibration curve samples (e.g., ten-point 0.01–25 µM series)

## Outputs

- Retention-time-scheduled transition method (1-minute window centered on measured RTs)
- Extracted ion chromatograms with integrated peak areas per analyte and internal standard
- CSV of per-analyte peak areas, analyte-to-internal-standard ratios, and calculated concentrations

## How to apply

Import a transition list into Skyline containing the target molecules (e.g., 15 arginine/polyamine-pathway analytes such as Arg, Orn, SAM, SAH, Spm, GABA, MTA, N1-/N8-AcSpd) paired with heavy-labeled internal standards, loading vendor raw files directly (vendor reading is provided through the ProteoWizard pwiz libraries). Export a retention-time-scheduled acquisition method using a 1-minute window centered on previously measured RTs for the ion-pairing method (e.g., Acquity UPLC/Xevo TQ-S with PFHA). Extract ion chromatograms, integrate peaks with Skyline's small-molecule peak picking, then calibrate against an external or stable-isotope dilution curve — e.g., a ten-point 0.01–25 µM series — to compute per-sample concentrations. Peak-picking, chromatographic alignment, and transition selection have been adapted to small-molecule data, and collision energies can be optimized under Settings→Transition Settings→Prediction. Export per-analyte peak areas, analyte/internal-standard ratios, and calculated concentrations to a CSV for downstream use.

## Related tools

- **Skyline** (Primary tool: imports transition lists, performs RT-scheduled small-molecule peak picking and integration, calibrated quantification, and method export) — https://github.com/ProteoWizard/pwiz-cpp
- **ProteoWizard/pwiz** (Underlying open-source libraries that enable direct reading of many vendor raw data formats into Skyline) — https://github.com/ProteoWizard/pwiz
- **Panorama** (Web-based repository for sharing Skyline documents and targeted methods/data) — https://panoramaweb.org/

## Evaluation signals

- Output concentration table contains rows for all 15 named polyamine/arginine-pathway analytes across every sample matrix (e.g., mouse urine and human brain tissue)
- No missing transitions: every target and heavy-labeled internal standard has an integrated chromatographic peak
- Calibration-curve fit is monotonic across the full ten-point 0.01–25 µM range and passes Skyline's standard QC
- Per-analyte peak areas, analyte/internal-standard ratios, and concentrations are all present in the exported CSV (e.g., polyamine_concentrations.csv)
- Exported method RT windows are 1-minute wide and centered on the measured retention times from the prior analysis

## Limitations

- For latest-generation or vendor-specific workflows, Skyline may lag behind commercial software until support is implemented
- No single tool is best for all workflows; Skyline's GC-MS and ion-mobility small-molecule support is still expanding
- Small-molecule fragmentation library workflows (METLIN, MassBank, HMDB, NIST) are future work, not yet available

## Evidence
