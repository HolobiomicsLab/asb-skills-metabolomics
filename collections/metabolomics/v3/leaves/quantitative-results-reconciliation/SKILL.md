---
name: quantitative-results-reconciliation
description: Use when a deposited Skyline small-molecule document template and reference
  quantitative results exist and raw data from a different MS vendor (Agilent, Bruker,
  Shimadzu, Thermo vs. the original Sciex/Waters) must be shown to yield consistent
  quantitative values.
when_to_use_negative:
- The analysis goal is untargeted feature-finding, alignment, and identification (the
  XCMS/MZmine/MS-DIAL paradigm), not targeted extraction and quantification of known
  transitions
- The workflow depends on small-molecule fragmentation libraries (METLIN, MassBank,
  HMDB, NIST), which Skyline does not yet implement
- The data are GC-MS or ion mobility acquisitions, for which Skyline support was still
  ongoing at publication time
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Open the deposited document template, import the non-report vendor's raw data,
    inspect EICs and peak integration, and export the quantitative CSV report for
    reconciliation against the reference document
  repo: github.com/ProteoWizard/pwiz-cpp
- name: Panorama Public
  role: Repository from which the Skyline document templates, reference quantitative
    results, and deposited vendor raw data files are obtained
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:caafe9ddda21df3211f45d0c6e702c05cd66f0cbe6951932614dc5e247e69763
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama Public
  - Skyline
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# quantitative-results-reconciliation

## Summary

Validates that a Skyline small-molecule quantitative workflow (e.g., the Biocrates p180 amino acid assay or a polyamine panel) transfers to mass spectrometer data from a vendor other than the one used in the original report, by reconciling exported peak areas and concentrations against the reference document from Panorama Public.

## When to use

Use when a deposited Skyline small-molecule document template and reference quantitative results exist and raw data from a different MS vendor (Agilent, Bruker, Shimadzu, Thermo vs. the original Sciex/Waters) must be shown to yield consistent quantitative values.

## When NOT to use

- The analysis goal is untargeted feature-finding, alignment, and identification (the XCMS/MZmine/MS-DIAL paradigm), not targeted extraction and quantification of known transitions
- The workflow depends on small-molecule fragmentation libraries (METLIN, MassBank, HMDB, NIST), which Skyline does not yet implement
- The data are GC-MS or ion mobility acquisitions, for which Skyline support was still ongoing at publication time

## Inputs

- Skyline document template (.sky) for a small-molecule assay (e.g., Biocrates p180 amino acids, polyamine panel, purine/pyrimidines, HRMS lipids) deposited on Panorama Public
- Reference quantitative results from the original Skyline document
- Vendor raw LC-MS data files from Agilent, Bruker, Shimadzu, or Thermo instruments (readable via ProteoWizard vendor readers)

## Outputs

- Skyline document with imported cross-vendor raw data and extracted ion chromatograms per transition
- CSV report of quantitative results (peak areas, concentrations where calibration is defined)
- Reconciliation verdict: whether cross-vendor peak areas/concentrations match the reference document within expected instrument-specific variability

## How to apply

Download the deposited Skyline document template (p180 kit assay or polyamine assay) and its reference quantitative results from Panorama Public, then open the template in Skyline so the small-molecule transition list, retention-time scheduling, and peak-picking settings are applied as configured. Obtain vendor raw data files from the non-report vendor (Agilent, Bruker, Shimadzu, or Thermo) — these are readable through ProteoWizard's vendor raw data readers on which Skyline is built. Import the raw files into the template and inspect extracted ion chromatograms to confirm transitions are correctly extracted and that integrated peak boundaries match the reference document; retention-time-scheduled methods in the paper use 1-minute windows centered on measured retention times. Export the quantitative results (peak areas, and concentrations where external calibration or stable-isotope dilution is defined) as a CSV report from Skyline. Accept the run when the same transitions yield peak areas/concentrations consistent with the reference document within instrument-specific cross-vendor variability.

## Related tools

- **Skyline** (Open the deposited document template, import the non-report vendor's raw data, inspect EICs and peak integration, and export the quantitative CSV report for reconciliation against the reference document) — github.com/ProteoWizard/pwiz-cpp
- **Panorama Public** (Repository from which the Skyline document templates, reference quantitative results, and deposited vendor raw data files are obtained) — https://panoramaweb.org/

## Evaluation signals

- Every transition in the template's small-molecule transition list yields an extracted ion chromatogram in the imported cross-vendor raw data with no missing entries
- Integrated peak boundaries in the cross-vendor document match the boundaries in the reference Panorama Public document
- Exported CSV peak areas/concentrations fall within the instrument-specific variability expected for cross-vendor data relative to the reference document's reported values
- Retention-time scheduling holds: extracted peaks fall within the template's scheduled windows (1-minute windows centered on measured retention times in the published workflow)
- Positive/negative switching data (purine/pyrimidine workflow) still reconcile within a single Skyline project

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; support for GC-MS and all types of ion mobility small molecule data was still ongoing
- Cross-vendor quantitative agreement is judged against instrument-specific variability, so exact value identity is not expected

## Evidence
