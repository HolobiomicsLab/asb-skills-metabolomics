---
name: raw-mass-spectrometry-file-conversion
description: Use when vendor RAW LC-HRMS2/MS2 DDA files (e.g., Thermo .RAW from plant-extract
  or metabolomics runs) must be converted to an open format (mzML/mzXML) before MZmine
  processing or repository upload.
when_to_use_negative:
- The MS data are already in an open format (mzML/mzXML), e.g., a deposited dataset
  like MSV00008616 retrieved from MassIVE — re-conversion is unnecessary.
- The goal is feature detection, annotation, or molecular networking on an existing
  feature table — this skill is only the RAW-format migration step, not the MZmine
  or FBMN analysis.
- The vendor instrument produces a format MSConvert/ProteoWizard does not support
  without vendor libraries.
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: MSConvert
  role: ProteoWizard tool that performs the vendor RAW to open mzML/mzXML conversion,
    retaining MS1 and MS2 scan data
- name: MZmine
  role: Downstream feature-detection consumer of the converted files; loadability
    in MZmine (v2.53) validates the conversion
  repo: http://mzmine.github.io/
- name: MassIVE
  role: Repository where the converted MS data files are shared
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:0a3592db7842579cb0ee31c8ffd6338904b12955c9cb784a27856ac63276127a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSConvert
  - MZmine
  - MassIVE
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# raw-mass-spectrometry-file-conversion

## Summary

Converts vendor LC-HRMS2 RAW files (e.g., Thermo .RAW) into an open mass-spectrometry format (mzML/mzXML) using MSConvert from ProteoWizard, the first step of the ENPKG workflow before MZmine feature detection and MassIVE deposition.

## When to use

Use when vendor RAW LC-HRMS2/MS2 DDA files (e.g., Thermo .RAW from plant-extract or metabolomics runs) must be converted to an open format (mzML/mzXML) before MZmine processing or repository upload.

## When NOT to use

- The MS data are already in an open format (mzML/mzXML), e.g., a deposited dataset like MSV00008616 retrieved from MassIVE — re-conversion is unnecessary.
- The goal is feature detection, annotation, or molecular networking on an existing feature table — this skill is only the RAW-format migration step, not the MZmine or FBMN analysis.
- The vendor instrument produces a format MSConvert/ProteoWizard does not support without vendor libraries.

## Inputs

- Vendor Thermo RAW LC-MS2 files (DDA, MS1 + MS2 scans)
- Sample-wise folder structure of plant-extract acquisition files

## Outputs

- One converted open-format file per RAW input (.mzML or .mzXML), retaining MS1 and MS2 scans
- Converted files suitable for upload to a MassIVE repository

## How to apply

Run MSConvert from the ProteoWizard package (v3.0.10385 in the ENPKG study) on each vendor Thermo RAW file, retaining both MS1 and MS2 scan data from the DDA acquisition (here, the negative-ionization-mode branch of the 1,600 plant extracts data set). Choose an open output format consistent with the downstream pipeline — the article reports the sample MS data as .mzML files (with features' MS2 spectra exported as .mgf), while the traced conversion task describes mzXML; either way the file must remain loadable by MZmine (v2.53). Do not modify scans or filter them at this stage; conversion is a format migration, and all thresholds (e.g., the MZmine centroid mass detector noise level of 1.0E4) are applied downstream. The converted files are also the archival format: the article states the MS data files are uploaded to a MassIVE repository. Validate that every input RAW yields a converted file that exists and is loadable by the downstream feature-detection step.

## Related tools

- **MSConvert** (ProteoWizard tool that performs the vendor RAW to open mzML/mzXML conversion, retaining MS1 and MS2 scan data)
- **MZmine** (Downstream feature-detection consumer of the converted files; loadability in MZmine (v2.53) validates the conversion) — http://mzmine.github.io/
- **MassIVE** (Repository where the converted MS data files are shared)

## Examples

```
msconvert sample.RAW --mzML -o converted/
```

## Evaluation signals

- Every processed input RAW file yields a converted mzML/mzXML file that exists on disk.
- Each converted file is loadable by the downstream MZmine (v2.53) step without errors.
- MS1 and MS2 scan data are both present in the converted file (scan counts consistent with the DDA acquisition).
- Converted files are in a repository-shareable format, matching the article's practice of uploading MS data files to MassIVE.

## Limitations

- PI-mode LC-MS2 acquisition parameters for the reference data set were deferred to ref (14), so conversion parameters alone do not fully reproduce the acquisition context.
- The source material is internally inconsistent on the exact output format (finding states .mzML files; the traced conversion task states mzXML) — the practitioner must match whatever format the downstream MZmine configuration expects.
- Conversion does not itself apply quality filters (e.g., MZmine noise level 1.0E4, feature RT window 0.5–8.0 min); those belong to downstream feature-detection steps.

## Evidence

- [other] MSConvert conversion step (task workflow, section not labeled): [evidence span withheld — anchor status: not_found]
- [results] MS data deposition: [evidence span withheld — anchor status: not_found]
- [readme] ENPKG workflow input requirements: [evidence span withheld — anchor status: external_doc]
