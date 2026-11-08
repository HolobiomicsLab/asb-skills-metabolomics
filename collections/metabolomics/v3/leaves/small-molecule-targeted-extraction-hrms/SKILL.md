---
name: small-molecule-targeted-extraction-hrms
description: Use when you have HRMS full-scan LC-MS raw files (e.g., Waters Synapt
  G2 HDMS) for treated vs. control samples and a list of small-molecule precursor
  ions with measured m/z and retention times whose peak areas you need to extract
  and compare.
when_to_use_negative:
- 'The analysis is untargeted: you need MS1 feature-finding, nonlinear peak alignment,
  matching, and identification across runs — use XCMS, MZmine, or MS-DIAL instead.'
- You need fragment/transition-level (SRM/MRM) small-molecule quantification rather
  than precursor-ion extraction from full-scan HRMS data.
- The workflow depends on latest-generation or vendor-specific features Skyline does
  not yet support (e.g., GC-MS or ion mobility small-molecule data), where Skyline
  may lag behind commercial software.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Skyline
  role: 'Primary tool: creates the small-molecule document, imports the precursor-ion
    transition list, extracts full-width precursor chromatograms from HRMS full-scan
    data, peak-picks, and reports integrated peak areas for group comparison.'
  repo: github.com/ProteoWizard/pwiz
- name: ProteoWizard/pwiz-cpp
  role: Source repository containing the Skyline codebase (pwiz_tools/Skyline) that
    implements the small-molecule targeted extraction; builds via pwiz_tools\Skyline\build.bat.
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Web repository for sharing the resulting Skyline documents and targeted method/data.
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:760b37e043cc35436e81f578df96d5bfc6c06e1c59f97a5ac70e200abfaf6f34
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - ProteoWizard/pwiz-cpp
  - Skyline
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# small-molecule-targeted-extraction-hrms

## Summary

Use Skyline's small-molecule mode to extract precursor-ion chromatograms for known targets (e.g., DAG 30:0 and DAG 32:0) from high-resolution MS full-scan data and quantify differential abundance between treatment groups, as demonstrated on Waters Synapt G2 HDMS lipidomics runs of BT474 cells.

## When to use

Use when you have HRMS full-scan LC-MS raw files (e.g., Waters Synapt G2 HDMS) for treated vs. control samples and a list of small-molecule precursor ions with measured m/z and retention times whose peak areas you need to extract and compare.

## When NOT to use

- The analysis is untargeted: you need MS1 feature-finding, nonlinear peak alignment, matching, and identification across runs — use XCMS, MZmine, or MS-DIAL instead.
- You need fragment/transition-level (SRM/MRM) small-molecule quantification rather than precursor-ion extraction from full-scan HRMS data.
- The workflow depends on latest-generation or vendor-specific features Skyline does not yet support (e.g., GC-MS or ion mobility small-molecule data), where Skyline may lag behind commercial software.

## Inputs

- High-resolution MS full-scan LC-MS raw files (e.g., Waters Synapt G2 HDMS runs) from treated and control samples
- Transition/precursor-ion list defining small-molecule targets with measured m/z values and retention times (e.g., DAG 30:0, DAG 32:0)
- Skyline small-molecule document (.sky) configured for precursor-only extraction

## Outputs

- Extracted ion chromatograms with Skyline peak-picked integrations for each precursor-ion target
- Integrated peak areas per target per run, grouped by treatment condition
- Fold-change values comparing treated vs. control groups, recorded in a results table (results_table.csv)

## How to apply

Create a new Skyline small-molecule document and import the transition list so the targets (e.g., DAG 30:0 and DAG 32:0) are defined as precursor-ion targets with their measured retention times and m/z values. Configure the small-molecule transition settings to extract full-width precursor ion chromatograms from the HRMS full-scan data with no fragment transitions (precursor-only extraction). Import all treated and control raw files and let Skyline peak-pick the extracted ion chromatograms, checking that integrated peak areas align with the expected retention windows. Finally, compare integrated peak areas between groups (e.g., FASN-inhibitor treated vs. control BT474 cells) to confirm differential expression and record fold-change values. The rationale is Skyline's targeted paradigm of extracting known targets followed by detection and quantification, rather than untargeted feature-finding followed by alignment and identification.

## Related tools

- **Skyline** (Primary tool: creates the small-molecule document, imports the precursor-ion transition list, extracts full-width precursor chromatograms from HRMS full-scan data, peak-picks, and reports integrated peak areas for group comparison.) — github.com/ProteoWizard/pwiz
- **ProteoWizard/pwiz-cpp** (Source repository containing the Skyline codebase (pwiz_tools/Skyline) that implements the small-molecule targeted extraction; builds via pwiz_tools\Skyline\build.bat.) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Web repository for sharing the resulting Skyline documents and targeted method/data.) — https://panoramaweb.org/

## Examples

```
pwiz_tools\Skyline\build.bat
```

## Evaluation signals

- Integrated peak areas must align with the published retention windows for each target (e.g., DAG 30:0 and DAG 32:0) when peak-picked by Skyline.
- Direction of change between treated and control groups must match the published figure/report (decreased DAG abundance in FASN-inhibitor treated cells).
- Magnitude of change must reproduce the reported fold-change values for DAG 30:0/DAG 32:0.
- Extracted chromatograms should show clean, well-integrated precursor-ion peaks in the full-width extraction window, consistent with targeted HRMS extraction.
- Fold-change values are recorded in results_table.csv with treated and control groups clearly separated.

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support can be implemented.
- No single tool will be best for all workflows; expansion of support for GC-MS and all types of ion mobility small molecule data are still ongoing.
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented in Skyline.

## Evidence
