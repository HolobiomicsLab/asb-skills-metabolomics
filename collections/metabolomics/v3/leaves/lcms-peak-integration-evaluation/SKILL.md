---
name: lcms-peak-integration-evaluation
description: Use when targeted LC-MS data (e.g., HRMS full-scan or SRM runs from instruments
  like the Waters Synapt G2 HDMS) must have known small-molecule targets extracted
  as ion chromatograms, peak-picked, and integrated for quantification or between-group
  comparison.
when_to_use_negative:
- Non-targeted profiling where features are unknown — use feature-finding/alignment
  tools such as XCMS, MZmine, or MS-DIAL instead of target extraction
- Input is already a feature table or peak-area matrix with no raw vendor data to
  re-integrate
- GC-MS or ion mobility small-molecule workflows, for which Skyline support is still
  ongoing per the paper's discussion
tools:
- name: Skyline
  role: Extracts precursor-ion targets from HRMS full-scan data as full-width chromatograms,
    peak-picks, and integrates peak areas for quantification
  repo: github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web repository for sharing the resulting Skyline documents, methods, and processed
    data
  repo: https://panoramaweb.org/
- name: ProteoWizard
  role: Underlying library/toolset enabling vendor raw data file access and mzML support
    used by Skyline
  repo: https://github.com/ProteoWizard/pwiz-cpp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:a4aed4f3b6a30a4415ab6de34dd6c193e54d6aba240e28c2b9afa43c2ccc75bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - ProteoWizard
  - Skyline
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lcms-peak-integration-evaluation

## Summary

Targeted extraction, peak-picking, and integration of small-molecule LC-MS chromatograms in Skyline, as extended from proteomics to metabolomics for SRM and high-resolution (HRMS) data. It converts raw instrument files into integrated peak areas for known targets that can be compared across conditions.

## When to use

Use when targeted LC-MS data (e.g., HRMS full-scan or SRM runs from instruments like the Waters Synapt G2 HDMS) must have known small-molecule targets extracted as ion chromatograms, peak-picked, and integrated for quantification or between-group comparison.

## When NOT to use

- Non-targeted profiling where features are unknown — use feature-finding/alignment tools such as XCMS, MZmine, or MS-DIAL instead of target extraction
- Input is already a feature table or peak-area matrix with no raw vendor data to re-integrate
- GC-MS or ion mobility small-molecule workflows, for which Skyline support is still ongoing per the paper's discussion

## Inputs

- Vendor raw LC-MS/MS data files (e.g., Waters Synapt G2 HDMS high-resolution full-scan runs; Skyline reads many vendor formats via ProteoWizard)
- Transition list (CSV) of small-molecule targets with precursor m/z and measured retention times
- Replicate raw files from treatment and control groups to be compared

## Outputs

- Extracted ion chromatograms (precursor-only, full-width) per target per run
- Skyline-integrated peak areas per replicate and target
- Group-level peak area comparisons / fold-change values (e.g., recorded in results_table.csv)
- Shareable Skyline document that can be published to Panorama

## How to apply

Create a new Skyline small-molecule document and import a transition list defining the targets of interest (precursor-ion m/z values and measured retention times, e.g., DAG 30:0 and DAG 32:0 from a lipidomics target list). Configure transition settings so chromatograms are extracted from HRMS full-scan data as full-width, precursor-only extracted ion chromatograms (no fragment transitions); when scheduling, use a retention-time window centered on the measured retention times (the paper used a 1-minute window). Import all raw vendor files and let Skyline peak-pick the extracted ion chromatograms, then inspect integration boundaries so the integrated peak areas align with the expected retention windows. The rationale is Skyline's targeted paradigm — extract known targets first, then detect and quantify — which is vendor-independent and molecule-centric, unlike non-targeted feature-finding/alignment tools such as XCMS, MZmine, and MS-DIAL.

## Related tools

- **Skyline** (Extracts precursor-ion targets from HRMS full-scan data as full-width chromatograms, peak-picks, and integrates peak areas for quantification) — github.com/ProteoWizard/pwiz
- **Panorama** (Web repository for sharing the resulting Skyline documents, methods, and processed data) — https://panoramaweb.org/
- **ProteoWizard** (Underlying library/toolset enabling vendor raw data file access and mzML support used by Skyline) — https://github.com/ProteoWizard/pwiz-cpp

## Evaluation signals

- Integrated peak areas for each target align with the known/published retention windows after automatic peak-picking
- Between-group peak area comparison reproduces the reported direction and magnitude of change (e.g., decreased DAG abundance in inhibitor-treated cells) from the published figure
- Fold-change values computed from integrated areas are recorded and consistent across replicate injections
- Extraction settings verified as precursor-only, full-width chromatograms from HRMS full-scan data (no unintended fragment transitions included)

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool is best for all workflows; support for GC-MS and all types of ion mobility small-molecule data is still ongoing
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented in Skyline

## Evidence
