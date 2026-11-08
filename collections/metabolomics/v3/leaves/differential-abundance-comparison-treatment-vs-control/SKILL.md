---
name: differential-abundance-comparison-treatment-vs-control
description: Use when HRMS full-scan LC-MS runs from treated vs. control samples need
  targeted re-extraction of known precursor ion m/z targets and a fold-change/direction
  comparison against a published result.
when_to_use_negative:
- Input is already a feature table from untargeted processing (e.g. XCMS/MZmine/MS-DIAL
  feature-finding output) rather than vendor raw HRMS files needing targeted extraction
- Data are GC-MS or latest-generation ion mobility small molecule data, where Skyline
  support is still ongoing and it may lag behind commercial software
- The goal is discovery of unknown differential features rather than confirming differential
  expression of known targets with measured m/z and retention times
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Small-molecule document creation, precursor ion target import, full-width
    precursor chromatogram extraction from HRMS full-scan data, peak picking and integration,
    and treated-vs-control area comparison
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: ProteoWizard
  role: Underlying open-source framework providing vendor raw data access and mzML
    support for the raw files processed in Skyline
  repo: https://github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web-based repository for sharing the Skyline documents produced by this analysis
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:0f00cd27fe5e51bb5c299a9339402c8e7cae5935f82ce9f29b75da7a48f8db1e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - ProteoWizard
  - Skyline
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# differential-abundance-comparison-treatment-vs-control

## Summary

Confirms differential abundance of specific small-molecule targets (e.g. DAG 30:0 and DAG 32:0) between FASN-inhibitor treated and control BT474 cells by extracting full-width precursor ion chromatograms from Synapt G2 HDMS HRMS full-scan data in Skyline and comparing integrated peak areas across groups.

## When to use

Use when HRMS full-scan LC-MS runs from treated vs. control samples need targeted re-extraction of known precursor ion m/z targets and a fold-change/direction comparison against a published result.

## When NOT to use

- Input is already a feature table from untargeted processing (e.g. XCMS/MZmine/MS-DIAL feature-finding output) rather than vendor raw HRMS files needing targeted extraction
- Data are GC-MS or latest-generation ion mobility small molecule data, where Skyline support is still ongoing and it may lag behind commercial software
- The goal is discovery of unknown differential features rather than confirming differential expression of known targets with measured m/z and retention times

## Inputs

- Vendor raw HRMS full-scan LC-MS files (Waters Synapt G2 HDMS) from FASN-inhibitor treated and control BT474 lipidomics runs
- Precursor ion transition/transition list with target m/z values and measured retention times for the molecules of interest (e.g. DAG 30:0, DAG 32:0)
- Previously published peak areas or figure values for the target molecules (reference for direction and magnitude of change)

## Outputs

- Skyline-extracted integrated peak areas for each target in every treated and control raw file
- Fold-change values for treated vs. control groups recorded in results_table.csv
- Extracted ion chromatograms with peak integration confirmable against published retention windows

## How to apply

Create a new Skyline small-molecule document and import a transition list containing the known targets (e.g. DAG 30:0, DAG 32:0) with their measured retention times and m/z values, following the Skyline paradigm of extracting known targets followed by detection and quantification. Configure small-molecule transition settings for precursor-only extraction of full-width chromatograms from HRMS full-scan data (no fragment transitions). Import all treated and control raw files and let Skyline peak-pick the extracted ion chromatograms, verifying integrated peak areas align with published retention windows. Finally, compare integrated peak areas between treatment groups to confirm the reported direction and magnitude of change (decreased DAG abundance in inhibitor-treated cells) and record fold-change values in results_table.csv.

## Related tools

- **Skyline** (Small-molecule document creation, precursor ion target import, full-width precursor chromatogram extraction from HRMS full-scan data, peak picking and integration, and treated-vs-control area comparison) — https://github.com/ProteoWizard/pwiz-cpp
- **ProteoWizard** (Underlying open-source framework providing vendor raw data access and mzML support for the raw files processed in Skyline) — https://github.com/ProteoWizard/pwiz
- **Panorama** (Web-based repository for sharing the Skyline documents produced by this analysis) — https://panoramaweb.org/

## Evaluation signals

- Integrated peak areas for each target fall within the published retention windows for DAG 30:0 and DAG 32:0
- Fold-change direction (decreased DAG abundance in FASN-inhibitor treated cells) matches the previously reported result
- Fold-change magnitude is comparable to the values shown in the published figure/report
- Peak picking/integration in Skyline completes for all imported treated and control raw files without missing targets
- results_table.csv contains fold-change entries for both DAG 30:0 and DAG 32:0

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; expansion of GC-MS and all types of ion mobility small molecule data support is ongoing
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented, limiting compound confirmation beyond precursor extraction

## Evidence
