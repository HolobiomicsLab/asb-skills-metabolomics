---
name: proteowizard-msconvert-parameterization
description: Use when the input consists of vendor Thermo RAW LC-HRMS2 files (negative
  or positive ionization mode) that must be converted to open formats before MZmine
  processing or repository upload.
when_to_use_negative:
- The input files are already in an open format (mzML/mzXML) — reconversion only risks
  loss or duplication
- The goal is feature detection, alignment, or .mgf MS2 spectrum export — those are
  performed by the downstream MZmine step, not by MSConvert
- The spectra require centroiding, noise filtering, or RT-window filtering — the ENPKG
  pipeline applies those filters later (e.g., feature filtering to RT 0.5–8.0 min
  in MZmine), not during format conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConvert (ProteoWizard)
  role: Converts vendor Thermo RAW LC-MS2 files to open mzXML/mzML format while retaining
    MS1 and MS2 scan data
- name: MZmine
  role: Downstream consumer of the converted files; performs feature detection and
    export (Figure 2B, step 1), and serves as the loadability validation target for
    the conversion
  repo: http://mzmine.github.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:1e314b4257d2eed0898de1ee53c104e08677487aa16c6d7662f9307898e652d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSConvert
  - MSConvert (ProteoWizard)
  - MZmine
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# proteowizard-msconvert-parameterization

## Summary

Parameterizes MSConvert (ProteoWizard) to convert vendor Thermo RAW LC-MS2 files from the ENPKG sample-centric workflow into open interchange formats (mzXML/mzML) that retain both MS1 and MS2 scans, so downstream MZmine feature detection and MassIVE sharing can proceed.

## When to use

Use when the input consists of vendor Thermo RAW LC-HRMS2 files (negative or positive ionization mode) that must be converted to open formats before MZmine processing or repository upload.

## When NOT to use

- The input files are already in an open format (mzML/mzXML) — reconversion only risks loss or duplication
- The goal is feature detection, alignment, or .mgf MS2 spectrum export — those are performed by the downstream MZmine step, not by MSConvert
- The spectra require centroiding, noise filtering, or RT-window filtering — the ENPKG pipeline applies those filters later (e.g., feature filtering to RT 0.5–8.0 min in MZmine), not during format conversion

## Inputs

- Vendor Thermo RAW LC-HRMS2 files (one per sample, e.g. from the 1,600 plant extracts data set, negative ionization mode branch)
- Sample-wise folder organization of the ENPKG data set (01_enpkg_data_organization scripts)

## Outputs

- Open-format converted MS files (mzXML per the workflow card; .mzML as the shared raw format) retaining MS1 and MS2 scan data
- Converted files that are loadable by MZmine (v2.53) for downstream feature detection and .mgf MS2 export

## How to apply

Load the vendor Thermo RAW LC-MS2 files (e.g., the 1,600 plant extracts data set in the negative ionization mode branch of the ENPKG workflow) and run MSConvert from the ProteoWizard package (v3.0.10385) to convert each RAW file into an open format, retaining both MS1 and MS2 scan data for subsequent feature detection. The converted files (mzML, together with the features' MS2 spectra exported as .mgf by MZmine) are the sample's raw MS data representation, which the ENPKG workflow uploads to a MassIVE repository so each sample can be processed individually. Process one RAW file per sample; the conversion precedes MZmine (v2.53) feature detection, so format fidelity for MS1 and MS2 levels is the key decision point. Do not filter or transform spectra at this stage — filtering (e.g., feature filtering on RT 0.5–8.0 min) happens downstream in MZmine.

## Related tools

- **MSConvert (ProteoWizard)** (Converts vendor Thermo RAW LC-MS2 files to open mzXML/mzML format while retaining MS1 and MS2 scan data)
- **MZmine** (Downstream consumer of the converted files; performs feature detection and export (Figure 2B, step 1), and serves as the loadability validation target for the conversion) — http://mzmine.github.io/

## Examples

```
msconvert <sample>.raw --mzXML -o converted/  # ProteoWizard v3.0.10385, MS1+MS2 retained for MZmine v2.53
```

## Evaluation signals

- Every processed input RAW file yields a converted mzXML file that exists and is loadable by the downstream MZmine (v2.53) step
- Both MS1 and MS2 scan data are present in each converted file (required by the subsequent feature-detection and MS2-export steps)
- One converted file per sample, consistent with the sample-wise folder architecture of the ENPKG data organization
- Converted files are suitable for upload to the MassIVE repository together with the features' MS2 spectra as .mgf files

## Limitations

- The conversion step is vendor- and instrument-bound (Thermo RAW files in this study); PI-mode LC-MS2 acquisition parameters are deferred to a prior reference (ref 14), so PI-mode RAW handling is not fully specified in this paper
- The workflow was designed for LC-MS/MS DDA metabolomics data in positive and/or negative ionization modes; other acquisition modes may need revalidation
- The README states the example dataset 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset', so input organization must match the ENPKG sample-wise layout

## Evidence

- [other] Run MSConvert from the ProteoWizard package (v3.0.10385) to convert each RAW file into open mzXML format, retaining both MS1 and MS2 scan data for subsequent feature detection: [evidence span withheld — anchor status: not_found]
- [other] In the ENPKG workflow, each sample's raw LC-HRMS2 data are stored as .mzML files (together with the features' MS2 spectra as .mgf files), and these MS data files are then uploaded to a MassIVE repository.: [evidence span withheld — anchor status: not_found]
- [readme] The Experimental Natural Products Knowledge Graph workflow aims at integrating experimental LC-MS/MS DDA metabolomics data into a Wikidata-connected knowledge graph.: [evidence span withheld — anchor status: external_doc]
