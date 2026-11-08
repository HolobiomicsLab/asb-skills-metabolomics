---
name: small-molecule-targeted-quantification
description: Use when you have vendor raw LC-MS data (e.g. Sciex, Waters, Agilent,
  Bruker, Shimadzu, Thermo) for known small-molecule targets (p180 amino acids, polyamines,
  purine/pyrimidines, HRMS lipids) and need calibrated peak-area quantification with
  a transition list and retention-time scheduling.
when_to_use_negative:
- For untargeted/nontargeted metabolomics feature-finding workflows, which start with
  feature detection in MS1 spectra followed by alignment and identification using
  tools like XCMS, MZmine, or MS-DIAL
- For GC-MS or ion mobility small molecule data, where Skyline support is still ongoing
- When you need fragmentation-library-driven identification from Metlin, MassBank,
  HMDB, or NIST — those workflows are not yet implemented in Skyline
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Skyline
  role: 'Primary tool: imports transition lists and vendor raw files, performs scheduled
    EIC extraction, peak picking, calibrated quantification, and CSV report export
    for small-molecule targeted assays'
  repo: github.com/ProteoWizard/pwiz
- name: ProteoWizard/pwiz
  role: Underlying library providing vendor raw-file readers and the Skyline codebase;
    build with pwiz_tools\Skyline\build.bat and follow the How To Build Skyline wiki
  repo: github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Repository for sharing the Skyline documents/templates and deposited raw data
    used as reference for this workflow
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:8f66cf03dfc8acf9ddc8d8ba3a57d344d8679ab46732e95b3ee4533cced00b92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - ProteoWizard/pwiz
  - Skyline
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# small-molecule-targeted-quantification

## Summary

Targeted quantitative analysis of small molecules/metabolites (SRM/MRM and HRMS) in Skyline: import transition lists and vendor raw files, apply scheduled extraction and peak picking, then export calibrated peak-area results. The paper demonstrates this on Biocrates p180 amino acids, polyamine, purine/pyrimidine, and HRMS lipid panels across multiple MS vendors.

## When to use

Use when you have vendor raw LC-MS data (e.g. Sciex, Waters, Agilent, Bruker, Shimadzu, Thermo) for known small-molecule targets (p180 amino acids, polyamines, purine/pyrimidines, HRMS lipids) and need calibrated peak-area quantification with a transition list and retention-time scheduling.

## When NOT to use

- For untargeted/nontargeted metabolomics feature-finding workflows, which start with feature detection in MS1 spectra followed by alignment and identification using tools like XCMS, MZmine, or MS-DIAL
- For GC-MS or ion mobility small molecule data, where Skyline support is still ongoing
- When you need fragmentation-library-driven identification from Metlin, MassBank, HMDB, or NIST — those workflows are not yet implemented in Skyline

## Inputs

- Vendor LC-MS raw data files (Sciex, Waters, Agilent, Bruker, Shimadzu, or Thermo formats readable through ProteoWizard/pwiz)
- Small-molecule transition list (precursor/product m/z, CE, RT) including heavy-labeled internal standards
- Skyline document template from Panorama Public (e.g. p180 kit assay, polyamine assay, purine/pyrimidine, HRMS lipid panel)
- Calibration information (stable-isotope dilution internal standards or external calibration curves)

## Outputs

- Extracted ion chromatograms with integrated peak areas per transition
- Quantitative results export (CSV report of peak areas and, where calibration is defined, concentrations)
- Retention-time-scheduled targeted acquisition method (1-minute window)

## How to apply

Download a Skyline document template from Panorama Public and import its small-molecule transition list, which includes molecules with heavy-labeled internal standards where available. Configure Transition Settings (including collision-energy prediction) and schedule the method to a 1-minute retention-time window centered on measured retention times. Import vendor raw data files so Skyline extracts ion chromatograms for each transition, then inspect peak picking and integration in the EICs. Quantify via stable-isotope dilution or external calibration to convert integrated peak areas into concentrations. Export a CSV report of peak areas/concentrations and compare against the reference document to confirm transitions were correctly extracted.

## Related tools

- **Skyline** (Primary tool: imports transition lists and vendor raw files, performs scheduled EIC extraction, peak picking, calibrated quantification, and CSV report export for small-molecule targeted assays) — github.com/ProteoWizard/pwiz
- **ProteoWizard/pwiz** (Underlying library providing vendor raw-file readers and the Skyline codebase; build with pwiz_tools\Skyline\build.bat and follow the How To Build Skyline wiki) — github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Repository for sharing the Skyline documents/templates and deposited raw data used as reference for this workflow) — https://panoramaweb.org/

## Examples

```
Skyline: File > Import > Transition List (p180/polyamine transitions with heavy internal standards) -> import vendor raw files (Sciex/Waters/Agilent/Bruker/Shimadzu/Thermo) -> File > Export > Report > CSV peak areas and concentrations.
```

## Evaluation signals

- Skyline EICs for each transition show a detected, correctly integrated peak at the scheduled retention time (1-minute window centered on expected RT)
- Exported CSV report contains peak areas for all expected transitions, including heavy-labeled internal standards for the p180 or polyamine panels
- Quantified concentrations match reference values from the Panorama Public document within instrument-specific variability for cross-vendor data
- Positive/negative switching purine/pyrimidine project produces transitions in both ionization modes within a single Skyline document
- Benchmarked peak-picking/integration results are consistent with the comparable instrument-vendor software tool used in the paper's comparison

## Limitations

- Skyline may lag behind commercial vendor software for latest-generation or vendor-specific workflows until support is implemented
- Support for GC-MS and all types of ion mobility small molecule data is still ongoing; no single tool is best for all workflows
- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented
- Proteomics-specific Panorama-enabled workflows (automated data import, AutoQC) are described as extended but not explicitly demonstrated for small molecules in this paper

## Evidence

- [other] The small molecule workflows demonstrated in this paper (p180 amino acids, polyamine panel, purine/pyrimidines, HRMS lipids) are stated to be functional in Skyline on Agilent, Bruker, Shimadzu, and Thermo instruments: [evidence span withheld — anchor status: not_found]
