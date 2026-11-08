---
name: lc-srm-chromatogram-peak-integration
description: Use when vendor raw LC-SRM/LC-MS files must be converted into quantified
  analyte concentrations via scheduled SRM transition extraction, peak picking, and
  isotope-dilution or external calibration in Skyline.
when_to_use_negative:
- The analysis is untargeted/nontargeted feature finding rather than extraction of
  known targets — use XCMS, MZmine, or MS-DIAL instead
- The data are GC-MS or ion mobility small molecule acquisitions, where Skyline support
  is still ongoing
- A vendor-specific latest-generation workflow is required where Skyline may lag behind
  commercial software
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Skyline
  role: Open-source tool used to import transition lists, extract ion chromatograms,
    pick and integrate peaks, and calibrate small-molecule SRM quantification
  repo: github.com/ProteoWizard/pwiz
- name: ProteoWizard (pwiz)
  role: Provides vendor raw file reading and mzML conversion underlying Skyline's
    data import
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Web repository for sharing Skyline documents and targeted methods/data
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:03f60131b1d607f84c46dee14fcaa12ea65243727061cafa6e4959965dccf0cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - ProteoWizard (pwiz)
  - Skyline
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# LC-SRM chromatogram peak integration

## Summary

Integrate targeted LC-SRM ion chromatograms in Skyline to quantify small-molecule analytes, using retention-time-scheduled transition lists, heavy-labeled internal standards, and calibration curves to produce per-analyte concentrations.

## When to use

Use when vendor raw LC-SRM/LC-MS files must be converted into quantified analyte concentrations via scheduled SRM transition extraction, peak picking, and isotope-dilution or external calibration in Skyline.

## When NOT to use

- The analysis is untargeted/nontargeted feature finding rather than extraction of known targets — use XCMS, MZmine, or MS-DIAL instead
- The data are GC-MS or ion mobility small molecule acquisitions, where Skyline support is still ongoing
- A vendor-specific latest-generation workflow is required where Skyline may lag behind commercial software

## Inputs

- Vendor raw LC-SRM data files (e.g., Waters Xevo TQ-S, multiple MS vendor formats readable via ProteoWizard)
- Retention-time-scheduled SRM transition list with target analytes and heavy-labeled internal standards
- Calibration curve standards (e.g., ten-point 0.01–25 µM series)

## Outputs

- Per-analyte peak areas and light/heavy internal-standard peak area ratios
- Calculated analyte concentrations per sample (e.g., polyamine_concentrations.csv)
- Extracted ion chromatograms with integrated peaks and QC status

## How to apply

Import a transition list of the target analytes (e.g., the 15 arginine/polyamine-pathway compounds with their heavy-labeled internal standards) into Skyline, then export a retention-time-scheduled method using a 1-minute window centered on measured RTs, as with the Acquity UPLC/Xevo TQ-S PFHA ion-pairing assay. Load the vendor raw files, extract ion chromatograms for each transition pair, and integrate peaks with Skyline's small-molecule peak picking. Quantify by stable-isotope dilution (peak area ratios to heavy internal standards) and/or external calibration against a multi-point curve (e.g., 0.01–25 µM ten-point series). Export per-analyte peak areas, light/heavy ratios, and calculated concentrations to a table such as polyamine_concentrations.csv. Validate that all targets are present, transitions are not missing, and calibration fits are monotonic.

## Related tools

- **Skyline** (Open-source tool used to import transition lists, extract ion chromatograms, pick and integrate peaks, and calibrate small-molecule SRM quantification) — github.com/ProteoWizard/pwiz
- **ProteoWizard (pwiz)** (Provides vendor raw file reading and mzML conversion underlying Skyline's data import) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Web repository for sharing Skyline documents and targeted methods/data) — https://panoramaweb.org/

## Examples

```
Build Skyline from the ProteoWizard repo: pwiz_tools\Skyline\build.bat  (then import the polyamine transition list and vendor raw files into Skyline-daily for RT-scheduled SRM integration)
```

## Evaluation signals

- Output table contains rows for all 15 target analytes (e.g., GABA, arginine, Orn, SAM, SAH, Spm, acetylated spermidines, MTA) in every matrix
- No missing transitions for any analyte across imported runs
- Calibration-curve fits are monotonic and pass Skyline's standard QC across the 0.01–25 µM range
- Peak integrations reviewed in extracted-ion-chromatogram view show correct peak boundaries and consistent retention times within the 1-minute scheduled window
- Light/heavy peak area ratios yield concentrations consistent with known matrix abundances (e.g., GABA ~800 nmol/g in human brain; Arg 4.36 µmol/mmol creatinine in mouse urine)

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- GC-MS and all types of ion mobility small molecule data support is still ongoing; no single tool will be best for all workflows
- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented

## Evidence
