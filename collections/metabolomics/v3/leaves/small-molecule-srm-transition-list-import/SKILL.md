---
name: small-molecule-srm-transition-list-import
description: Use when the input is a custom small-molecule SRM transition list (e.g.,
  purine/pyrimidine targets with heavy internal standards) and raw LC-SRM files from
  polarity-switching acquisitions on instruments like a SCIEX 5500.
when_to_use_negative:
- The analysis is untargeted feature-finding from MS1 spectra followed by alignment
  and identification — use a nontargeted tool like XCMS, MZmine, or MS-DIAL instead
- The data are high-resolution MS (HRMS) extractions or GC-MS/ion mobility small-molecule
  data, which require Skyline's separate HRMS support or are still ongoing work
- The input is a peptide-centric proteomics method where sequence-specific transitions
  are expected rather than small-molecule precursor/product pairs
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Skyline
  role: Imports the small-molecule SRM transition list and polarity-switching raw
    files into a single document, picks peaks for both polarities, and performs calibrated
    quantification
  repo: github.com/ProteoWizard/pwiz
- name: ProteoWizard/pwiz
  role: Provides the vendor raw-data reader framework that lets Skyline read vendor
    LC-SRM formats directly
  repo: github.com/ProteoWizard/pwiz-cpp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:d4fc5d24d1e652d0ce8f1bca49f2d1ef216092c4a4205da36d8a978216fdaaa3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ProteoWizard/pwiz
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# small-molecule-srm-transition-list-import

## Summary

Import a small-molecule SRM/MRM transition list (with heavy-labeled internal standards) into a Skyline document and process vendor raw data from rapid positive/negative polarity-switching acquisitions, so that 100+ transitions across both ionization modes are handled in a single project.

## When to use

Use when the input is a custom small-molecule SRM transition list (e.g., purine/pyrimidine targets with heavy internal standards) and raw LC-SRM files from polarity-switching acquisitions on instruments like a SCIEX 5500.

## When NOT to use

- The analysis is untargeted feature-finding from MS1 spectra followed by alignment and identification — use a nontargeted tool like XCMS, MZmine, or MS-DIAL instead
- The data are high-resolution MS (HRMS) extractions or GC-MS/ion mobility small-molecule data, which require Skyline's separate HRMS support or are still ongoing work
- The input is a peptide-centric proteomics method where sequence-specific transitions are expected rather than small-molecule precursor/product pairs

## Inputs

- Small-molecule SRM transition list (precursor/product m/z, polarity, heavy-labeled internal standard entries, e.g., from IROA MSMLS)
- Vendor raw LC-SRM data files from polarity-switching acquisitions (e.g., SCIEX 5500 .wiff)
- Published assay retention times / reference quantitative values (e.g., Table S1) for verification

## Outputs

- Skyline document with all positive- and negative-mode SRM transitions and picked/integrated chromatographic peaks
- Retention-time-scheduled SRM method export (1-minute window centered on measured retention times)
- Calibrated quantitative concentration table for each target relative to heavy internal standards

## How to apply

Build a Skyline molecule document by importing the transition list of small-molecule targets, including heavy-labeled internal standards (e.g., IROA MSMLS standards), as the molecule names and precursor/product ion pairs. Import the vendor raw LC-SRM files (e.g., SCIEX 5500 .wiff) directly into the same Skyline document, then configure transition settings so polarity switching between positive and negative mode transitions is handled within one project rather than split across two. Integrate the extracted ion chromatograms for each transition, checking peak integration and retention-time scheduling consistent with the published assay; retention-time-scheduled methods can be exported with a 1-minute window centered on measured retention times. For quantification, use calibrated quantification relative to the heavy internal standards and export results for comparison against reference values (e.g., Table S1).

## Related tools

- **Skyline** (Imports the small-molecule SRM transition list and polarity-switching raw files into a single document, picks peaks for both polarities, and performs calibrated quantification) — github.com/ProteoWizard/pwiz
- **ProteoWizard/pwiz** (Provides the vendor raw-data reader framework that lets Skyline read vendor LC-SRM formats directly) — github.com/ProteoWizard/pwiz-cpp

## Evaluation signals

- All transitions from the imported list (e.g., 112 SRM transitions under both ionization modes) appear in the Skyline document and are matched to chromatograms
- Positive- and negative-mode transitions are processed in a single Skyline project with peaks picked for both polarities
- Total acquisition cycle time reported by the instrument method remains ~1.3 seconds after scheduling
- Calculated concentrations relative to heavy internal standards reproduce the values reported in the published reference table (e.g., Table S1)
- Peak integration boundaries are manually inspectable per transition and consistent with the published assay's retention-time scheduling

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented in Skyline
- No single tool will be best for all workflows; GC-MS and ion mobility small-molecule data support is still ongoing

## Evidence

- [results] 112 SRM transitions, single acquisition: [evidence span withheld — anchor status: not_found]
