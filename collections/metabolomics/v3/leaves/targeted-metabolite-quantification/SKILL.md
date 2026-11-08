---
name: targeted-metabolite-quantification
description: Use when you have vendor LC-MS/MS raw files for a defined panel of small-molecule
  targets (with or without heavy-labeled internal standards) and need calibrated concentrations
  rather than a discovery feature table.
when_to_use_negative:
- You have unlabeled full-scan data needing unsupervised feature finding, alignment,
  and annotation — a non-targeted workflow (XCMS, MZmine, MS-DIAL) is the better fit,
  and the paper itself cautions that no single tool is best for all workflows.
- Input is already a curated feature/quantification table rather than raw vendor acquisitions
  — Skyline's chromatogram extraction and peak picking would be redundant.
- You need GC-MS or ion-mobility small-molecule workflows today; the article states
  support for those is still ongoing, so Skyline may lag commercial software.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: Skyline
  role: 'Primary tool: imports the transition list, extracts ion chromatograms from
    vendor raw files, performs small-molecule peak picking, RT-scheduled method export,
    and calibrated quantification.'
  repo: https://github.com/ProteoWizard/pwiz
- name: ProteoWizard/pwiz
  role: Underlying open-source library/format layer that Skyline depends on; builds
    Skyline via pwiz_tools/Skyline/build.bat and supports vendor raw-format reading.
  repo: https://github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web repository for sharing Skyline documents and targeted methods between
    labs and instrument platforms.
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:6e37f57e4c1fa769937465af2f7aab0e25092f9b5423d119dd936013188c5dc9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz
  tools:
  - Panorama
  - ProteoWizard/pwiz
  - Skyline
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# targeted-metabolite-quantification

## Summary

Quantify targeted metabolites (e.g., polyamines, amino acids, purines/pyrimidines) from LC-MS/MS raw data using Skyline's small-molecule workflows, which handle selected reaction monitoring (SRM) and HRMS extraction, retention-time scheduling, stable-isotope dilution, and calibrated quantification in a single vendor-independent project.

## When to use

Use when you have vendor LC-MS/MS raw files for a defined panel of small-molecule targets (with or without heavy-labeled internal standards) and need calibrated concentrations rather than a discovery feature table.

## When NOT to use

- You have unlabeled full-scan data needing unsupervised feature finding, alignment, and annotation — a non-targeted workflow (XCMS, MZmine, MS-DIAL) is the better fit, and the paper itself cautions that no single tool is best for all workflows.
- Input is already a curated feature/quantification table rather than raw vendor acquisitions — Skyline's chromatogram extraction and peak picking would be redundant.
- You need GC-MS or ion-mobility small-molecule workflows today; the article states support for those is still ongoing, so Skyline may lag commercial software.

## Inputs

- Vendor LC-MS/MS raw acquisition files (e.g., Waters Xevo TQ-S SRM runs) importable by Skyline
- Targeted transition list (analyte name, precursor/product m/z, charge, optional heavy-labeled internal standards)
- Calibration-curve samples and concentrations (e.g., ten-point 0.01–25 µM standard series)
- Measured retention times for scheduling (optional, for RT-scheduled method export)

## Outputs

- Per-analyte extracted ion chromatograms with integrated peak areas and analyte/internal-standard peak-area ratios
- Calculated metabolite concentrations per sample (e.g., polyamine_concentrations.csv)
- Retention-time-scheduled acquisition method (1-minute window centered on measured RTs)
- Skyline document shareable through Panorama (https://panoramaweb.org/)

## How to apply

Build a transition list naming each target analyte, its precursor/product m/z, charge, and any heavy-labeled internal standards, and import it into a Skyline document with the appropriate transition settings (precursor/product charge, m/z tolerance, and collision-energy prediction via Settings→Transition Settings→Prediction). Load the vendor raw files (e.g., Waters Xevo TQ-S SRM acquisitions), extract chromatograms, and let Skyline's small-molecule peak picking integrate the peaks; for scheduled methods, export a retention-time-scheduled method with a 1-minute window centered on the measured RTs. Apply quantification through stable-isotope dilution or an external calibration curve — for example a ten-point 0.01–25 µM curve — to convert integrated peak areas or analyte/internal-standard ratios into concentrations per sample. Decision points include validating that every transition is present and the calibration is monotonic before trusting downstream numbers; export per-analyte areas, ratios, and calculated concentrations (e.g., to a polyamine_concentrations.csv). Rationale: Skyline is molecule-centric and vendor-agnostic, so the same document supports SRM and high-resolution extraction and can be shared via Panorama for method translation between labs.

## Related tools

- **Skyline** (Primary tool: imports the transition list, extracts ion chromatograms from vendor raw files, performs small-molecule peak picking, RT-scheduled method export, and calibrated quantification.) — https://github.com/ProteoWizard/pwiz
- **ProteoWizard/pwiz** (Underlying open-source library/format layer that Skyline depends on; builds Skyline via pwiz_tools/Skyline/build.bat and supports vendor raw-format reading.) — https://github.com/ProteoWizard/pwiz
- **Panorama** (Web repository for sharing Skyline documents and targeted methods between labs and instrument platforms.) — https://panoramaweb.org/

## Examples

```
pwiz_tools\Skyline\build.bat  # or import a transition list into a Skyline document, set Settings→Transition Settings→Prediction, load vendor raw files, and export a 1-minute RT-scheduled SRM method
```

## Evaluation signals

- Every listed transition for all target analytes (e.g., 15 polyamine-pathway compounds) is present and integrated — no missing transitions in the Skyline document.
- Calibration-curve response is monotonic across the standard series and the fit passes Skyline QC; analyte/internal-standard ratios fall within the linear range (e.g., 0.01–25 µM).
- Exported concentration table contains a row for each named analyte and per-sample values consistent with expected matrix levels (e.g., GABA ~800 nmol/g in human brain, Arg 4.36 µmol/mmol creatinine in mouse urine).
- RT-scheduled method export uses a 1-minute window centered on the measured retention times and produces no scheduling conflicts across the analyte panel.
- Peak integrations are reviewed per sample and agree with Skyline's standard small-molecule peak-picking QC (no misplaced boundaries or missing heavy/light pairs).

## Limitations

- Skyline may lag behind commercial vendor software for latest-generation or vendor-specific workflows until support is implemented.
- Support for GC-MS and ion-mobility small-molecule data is still expanding; no single tool is best for all workflows.
- Fragmentation-library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented in Skyline.
- Proteomics-style automated data import and AutoQC are shown for peptides but not explicitly demonstrated for small molecules in this paper.

## Evidence
