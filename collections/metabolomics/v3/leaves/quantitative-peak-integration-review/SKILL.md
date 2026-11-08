---
name: quantitative-peak-integration-review
description: Use when quantitative peak areas or concentrations for targeted small-molecule
  metabolites (SRM, MRM, or HRMS-extracted transitions) must be validated or corrected
  in Skyline before reporting, e.g. polarity-switched purine/pyrimidine panels, Biocrates
  p180 amino acids, or polyamine assays.
when_to_use_negative:
- The goal is untargeted feature finding, alignment, and identification from MS1 spectra
  — use XCMS, MZmine, or MS-DIAL instead, since Skyline's paradigm here is extracting
  known targets followed by detection and quantification, not discovery workflows.
- Data are latest-generation or vendor-specific formats whose support has not yet
  been implemented in Skyline, so peak extraction itself may lag behind the vendor's
  own software.
- The analytes are unfragmented GC-MS or ion-mobility small-molecule datasets for
  which Skyline support is explicitly described as still ongoing rather than established.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: 'Main tool: imports vendor raw data, extracts ion chromatograms for each transition,
    supports manual re-integration of peak boundaries across positive/negative modes,
    and applies calibrated quantification to produce peak-area/concentration reports'
- name: Panorama (Panorama Public)
  role: Repository used to share the Skyline .sky documents and quantitative results
    so integration decisions can be cross-checked against a reference document
  repo: https://panoramaweb.org/
- name: ProteoWizard (pwiz)
  role: Upstream project housing Skyline under pwiz_tools/Skyline; provides the vendor
    raw-data reading layer that lets Skyline open files from multiple instrument vendors
    for chromatogram review
  repo: github.com/ProteoWizard/pwiz
- name: XCM
  role: Alternative used for untargeted feature finding when a peak-review step is
    not appropriate; contrast tool named in the article, not part of this skill's
    workflow
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:38b9f61b77f800c9f127c5f8cf96eade282c2c80a518bafe07915e0669acb4d0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama (Panorama Public)
  - ProteoWizard (pwiz)
  - Skyline
  - XCM
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quantitative-peak-integration-review

## Summary

A Skyline-centric review step for targeted small-molecule quantification in which extracted ion chromatograms (XICs) from SRM/PRM/HRMS acquisitions are inspected and re-integrated so that peak boundaries, transition coelution, and retention-time scheduling match the calibrated assay design before concentrations are exported.

## When to use

Use when quantitative peak areas or concentrations for targeted small-molecule metabolites (SRM, MRM, or HRMS-extracted transitions) must be validated or corrected in Skyline before reporting, e.g. polarity-switched purine/pyrimidine panels, Biocrates p180 amino acids, or polyamine assays.

## When NOT to use

- The goal is untargeted feature finding, alignment, and identification from MS1 spectra — use XCMS, MZmine, or MS-DIAL instead, since Skyline's paradigm here is extracting known targets followed by detection and quantification, not discovery workflows.
- Data are latest-generation or vendor-specific formats whose support has not yet been implemented in Skyline, so peak extraction itself may lag behind the vendor's own software.
- The analytes are unfragmented GC-MS or ion-mobility small-molecule datasets for which Skyline support is explicitly described as still ongoing rather than established.

## Inputs

- Vendor raw LC-SRM/LC-HRMS files from supported instruments (Sciex, Waters, Agilent, Bruker, Shimadzu, Thermo), e.g. SCIEX 5500 polarity-switching runs
- Skyline transition list for small-molecule targets with quantifier/qualifier transitions and heavy-labeled internal standards (e.g. IROA MSMLS heavy purines/pyrimidines)
- Retention-time-scheduled transition settings (window length, expected RTs) and calibration/quantification settings (stable-isotope dilution or external calibration curve)
- Reference Skyline document from Panorama Public for cross-checking expected peak areas/concentrations

## Outputs

- Reviewed and (where needed) manually re-integrated peak boundaries for each transition and molecule
- Recalculated integrated peak areas per transition per injection
- Calibrated concentrations of small-molecule compounds relative to heavy internal standards (e.g. Table S1-style values for purines/pyrimidines or p180 amino acid concentrations)
- Exported quantitative report (CSV) suitable for deposit alongside the .sky document on Panorama

## How to apply

Import vendor raw LC-SRM or LC-HRMS files into a Skyline document whose transition list already contains the small-molecule targets plus heavy-labeled internal standards (e.g. IROA MSMLS purines/pyrimidines or p180 amino acids). Skyline extracts each transition as an XIC; open the chromatogram view and check that integrated peak boundaries cover the expected retention time scheduled in the method (e.g. a 1-minute window centered on measured retention times) and that coeluting transitions for the same molecule peak together rather than being picked independently. Where automatic peak picking misplaces a boundary, drag or re-integrate manually so that quantifier and qualifier transitions and the heavy/light internal-standard pair share consistent integration, then let Skyline recalculate peak areas and the calibrated concentrations derived from stable-isotope dilution or external calibration. Positive and negative ionization mode transitions for a compound (as in the purine/pyrimidine SCIEX 5500 panel of 112 SRM transitions with ~1.3 s cycle time) remain in the same project, so review both polarities in one pass. Finish by exporting the corrected peak-area/concentration report and, where possible, comparing against a reference document on Panorama to confirm the integration reproduces published values.

## Related tools

- **Skyline** (Main tool: imports vendor raw data, extracts ion chromatograms for each transition, supports manual re-integration of peak boundaries across positive/negative modes, and applies calibrated quantification to produce peak-area/concentration reports)
- **Panorama (Panorama Public)** (Repository used to share the Skyline .sky documents and quantitative results so integration decisions can be cross-checked against a reference document) — https://panoramaweb.org/
- **ProteoWizard (pwiz)** (Upstream project housing Skyline under pwiz_tools/Skyline; provides the vendor raw-data reading layer that lets Skyline open files from multiple instrument vendors for chromatogram review) — github.com/ProteoWizard/pwiz
- **XCM** (Alternative used for untargeted feature finding when a peak-review step is not appropriate; contrast tool named in the article, not part of this skill's workflow)

## Examples

```
# In Skyline (GUI): File > Import > Results... -> select SCIEX 5500 polarity-switching .wiff files for the purine/pyrimidine .sky document; review chromatograms per transition; File > Export > Report > Quantitative+ -> peak areas & concentrations CSV
```

## Evaluation signals

- Every reviewed molecule's quantifier, qualifier, and heavy internal-standard transitions share consistent peak boundaries (coeluting integration rather than per-transition independent picking).
- Peak boundaries fall within the scheduled retention-time window defined in the method (e.g. the 1-minute window centered on measured retention times used for method export).
- Exported peak areas and concentrations reproduce the reference values in the published document (e.g. purine/pyrimidine Table S1) within instrument-specific variability when reprocessing a deposited dataset.
- Cycle time and transition count of the method are preserved on re-import (e.g. 112 SRM transitions across positive and negative modes with ~1.3 s cycle time for the purine/pyrimidine panel).
- The corrected .sky document and CSV report can be re-imported and re-processed in Skyline without integration changes, confirming reproducibility of the review.

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented, so some vendor raw formats may not yet be reviewable.
- No single tool will be best for all workflows: GC-MS and ion-mobility small-molecule data support in Skyline is still ongoing, so peak integration review for those modalities is out of scope.
- Fragmentation-library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented, so transition selection for review must come from existing transition lists rather than spectral libraries.
- Peptide-centric Panorama workflows such as automated data import and AutoQC have been extended to small molecules in principle, but this paper does not demonstrate them explicitly for small-molecule peak review.

## Evidence

- [abstract] SRM processing in Skyline: XIC extraction, integration review, polarity handling: [evidence span withheld — anchor status: not_found]
- [discussion] Cross-vendor applicability of the small-molecule workflows: [evidence span withheld — anchor status: not_found]
