---
name: lipidomics-precursor-ion-targeting
description: Use when specific lipid species must be confirmed or quantified as precursor-ion
  targets in HRMS full-scan lipidomics data (e.g., Waters Synapt G2 HDMS runs of FASN-inhibitor
  treated vs. control cells) rather than discovered by untargeted feature finding.
when_to_use_negative:
- When the goal is untargeted lipid discovery (MS1 feature-finding, alignment, identification)
  — use XCMS, MZmine, or MS-DIAL instead of targeted precursor extraction
- When fragment/transition-based quantification (e.g., SRM/MRM with product ions)
  is required — this skill is precursor-only extraction from full-scan HRMS data
- When the instrument data is GC-MS or ion-mobility small molecule data, where the
  article notes Skyline support is still ongoing and no single tool is best for all
  workflows
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: Skyline
  role: Small-molecule document creation, import of the precursor transition list,
    full-width precursor chromatogram extraction from HRMS full-scan data, peak picking,
    and area integration across treated/control runs
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Web-based repository for sharing the Skyline document and processed targeted
    data
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:bd5ab14ff117fd73990648d0e92d7125e301716061f774943b840f7de4ad7b74
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lipidomics-precursor-ion-targeting

## Summary

Targeted extraction and quantification of specific lipid species (e.g., diacylglycerols DAG 30:0 and DAG 32:0) as precursor-ion targets from high-resolution MS full-scan lipidomics data using Skyline's small-molecule mode, enabling differential expression testing between treatment groups without fragment transitions.

## When to use

Use when specific lipid species must be confirmed or quantified as precursor-ion targets in HRMS full-scan lipidomics data (e.g., Waters Synapt G2 HDMS runs of FASN-inhibitor treated vs. control cells) rather than discovered by untargeted feature finding.

## When NOT to use

- When the goal is untargeted lipid discovery (MS1 feature-finding, alignment, identification) — use XCMS, MZmine, or MS-DIAL instead of targeted precursor extraction
- When fragment/transition-based quantification (e.g., SRM/MRM with product ions) is required — this skill is precursor-only extraction from full-scan HRMS data
- When the instrument data is GC-MS or ion-mobility small molecule data, where the article notes Skyline support is still ongoing and no single tool is best for all workflows

## Inputs

- Vendor raw HRMS full-scan LC-MS files from lipidomics runs (e.g., Waters Synapt G2 HDMS acquisitions of treated and control samples)
- Precursor ion transition list specifying lipid species (m/z and measured retention times, e.g., DAG 30:0 and DAG 32:0)
- Skyline small-molecule document configuration (transition settings set for precursor-only extraction)

## Outputs

- Extracted ion chromatograms (full-width precursor XICs) per lipid target per run
- Integrated peak areas for each target in each treated/control file
- Fold-change comparison table of differential expression (e.g., results_table.csv)

## How to apply

Create a new small-molecule document in Skyline (v4.2 or later, which is molecule-centric and agnostic to molecule source) and import the lipid transition list — for the DAG 30:0/DAG 32:0 workflow, add each species as a precursor-ion target with its measured m/z and retention time. Configure transition settings to extract full-width precursor ion chromatograms from the HRMS full-scan data (precursor-only extraction, no fragment transitions), then import all treated and control raw files (Waters Synapt G2 HDMS runs of BT474 cells) and let Skyline peak-pick the extracted ion chromatograms. Verify that integrated peak areas align with the published retention windows. Rationale: the Skyline targeted paradigm — extracting known targets followed by detection and quantification — is vendor-independent and directly contrasts with non-targeted workflows that start with MS1 feature-finding and alignment. Finally, compare integrated peak areas between FASN-inhibitor treated and control groups to confirm the differential direction (decreased DAG abundance in inhibitor-treated cells) and record fold-change values in results_table.csv.

## Related tools

- **Skyline** (Small-molecule document creation, import of the precursor transition list, full-width precursor chromatogram extraction from HRMS full-scan data, peak picking, and area integration across treated/control runs) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Web-based repository for sharing the Skyline document and processed targeted data) — https://panoramaweb.org/

## Evaluation signals

- Integrated Skyline peak areas fall within the published retention windows for DAG 30:0 and DAG 32:0
- Extracted chromatograms are full-width precursor XICs with no fragment transitions (precursor-only extraction setting confirmed)
- Fold-change direction and magnitude of DAG 30:0/DAG 32:0 reproduce the differential expression shown in the published figure (decreased DAG abundance in FASN-inhibitor treated cells)
- All treated and control raw files are imported and peak-picked, with areas recorded in results_table.csv
- Peak areas are consistent between replicate treated and control injections (comparable peak-picking behavior across runs)

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows (including ion mobility) until support is implemented
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented, so spectral confirmation of lipid identity is limited
- No single tool will be best for all workflows; vendor-independent targeted extraction may underperform specialized vendor software for some instruments

## Evidence
