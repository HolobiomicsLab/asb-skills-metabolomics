---
name: lc-hrms-vendor-format-handling
description: Use when the pipeline starts from vendor-proprietary LC-HRMS2 acquisition
  files (e.g., Thermo RAW from UHPLC-HRMS2 DDA runs) and open, tool-readable mzML/mzXML
  files are needed for feature detection and repository sharing.
when_to_use_negative:
- The input data are already in an open format (mzML/mzXML) loadable by MZmine — re-conversion
  is unnecessary
- The task is feature detection or annotation on existing converted files (that is
  the downstream MZmine step, not format conversion)
- The data are not LC-HRMS2 acquisitions (e.g., already-processed feature quantification
  tables)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConvert (ProteoWizard)
  role: Performs the vendor RAW-to-open-format (mzML/mzXML) conversion, retaining
    MS1 and MS2 scan data; version v3.0.10385 used in the ENPKG workflow
- name: MZmine
  role: Downstream consumer of the converted files — performs feature detection and
    export (Figure 2B, step 1); conversion is validated by files being loadable in
    MZmine v2.53
  repo: http://mzmine.github.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:e9960a9283e50949278b2545a1ce115a12d8b1d7ed08891adbf3382ee96644c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSConvert (ProteoWizard)
  - MZmine
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lc-hrms-vendor-format-handling

## Summary

Converts vendor LC-HRMS2 RAW files (e.g., Thermo RAW) into open mzML/mzXML format using MSConvert from ProteoWizard (v3.0.10385), retaining MS1 and MS2 scan data so downstream tools like MZmine can perform feature detection in the ENPKG sample-centric workflow.

## When to use

Use when the pipeline starts from vendor-proprietary LC-HRMS2 acquisition files (e.g., Thermo RAW from UHPLC-HRMS2 DDA runs) and open, tool-readable mzML/mzXML files are needed for feature detection and repository sharing.

## When NOT to use

- The input data are already in an open format (mzML/mzXML) loadable by MZmine — re-conversion is unnecessary
- The task is feature detection or annotation on existing converted files (that is the downstream MZmine step, not format conversion)
- The data are not LC-HRMS2 acquisitions (e.g., already-processed feature quantification tables)

## Inputs

- Vendor Thermo RAW LC-MS2/MS1+MS2 DDA acquisition files (UHPLC-HRMS2, positive and/or negative ionization modes)
- Sample-wise folder organization with per-sample raw data (ENPKG 01_enpkg_data_organization layout)

## Outputs

- Converted open-format mzML files containing retained MS1 and MS2 scan data
- MS2 spectra exported as .mgf files
- Converted MS data files staged for upload to a MassIVE repository

## How to apply

Load the vendor Thermo RAW LC-MS2 files (in the ENPKG use case, the 1,600 plant extracts data set processed in the negative ionization mode branch) and run MSConvert from the ProteoWizard package (v3.0.10385) to convert each RAW file into an open mzML/mzXML file. Retain both MS1 and MS2 scan data, since the downstream MZmine (v2.53) step needs MS1 for feature detection (centroid mass detector, ADAP chromatogram builder, wavelets/ADAP deconvolution) and MS2 for annotation and molecular networking. Alongside the converted files, the features' MS2 spectra are exported as .mgf files. The resulting per-sample MS data files are then uploaded to a MassIVE repository so each sample's data is publicly addressable. Validation: every processed input RAW file yields a converted open-format file that exists and is loadable by MZmine.

## Related tools

- **MSConvert (ProteoWizard)** (Performs the vendor RAW-to-open-format (mzML/mzXML) conversion, retaining MS1 and MS2 scan data; version v3.0.10385 used in the ENPKG workflow)
- **MZmine** (Downstream consumer of the converted files — performs feature detection and export (Figure 2B, step 1); conversion is validated by files being loadable in MZmine v2.53) — http://mzmine.github.io/

## Examples

```
msconvert sample.raw --mzML
```

## Evaluation signals

- Every processed input RAW file yields a converted mzML/mzXML file that exists on disk
- Each converted file is loadable by the downstream MZmine (v2.53) step without import errors
- Converted files retain both MS1 and MS2 scan data (required for feature detection and MS2-based annotation)
- Per-sample .mgf files of MS2 spectra are exported alongside the converted files
- Converted MS data files are successfully uploaded to a MassIVE repository and receive a stable accession

## Limitations

- The conversion applies to vendor RAW formats of the specific instrument used (Thermo RAW in the demonstrated 1,600 plant extracts data set); other vendor formats may need equivalent handling
- Conversion is only the entry point of the workflow — it does not by itself detect features or annotate structures; PI-mode LC-MS2 acquisition parameters for one branch were deferred to a prior reference (ref 14)
- Downstream feature alignment across batches remains a known limitation the ENPKG design circumvents by processing each sample individually, not something the conversion step solves

## Evidence

- [other] MSConvert RAW conversion retaining MS1/MS2 scans: [evidence span withheld — anchor status: not_found]
- [results] MS data files uploaded to MassIVE: [evidence span withheld — anchor status: not_found]
- [methods] Sample-wise data organization for conversion and processing: "organized in a sample-wise folder architecture using scripts available at https://github.com/enpkg/enpkg_full/tree/main/01_enpkg_data_organization"
- [readme] Workflow input data per sample: [evidence span withheld — anchor status: external_doc]
- [results] Downstream feature detection consumes converted files: "feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine"
