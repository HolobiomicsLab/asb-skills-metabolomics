---
name: raw-ms-data-format-conversion
description: Use when raw vendor LC–MS acquisition files (Waters RAW) must be converted
  to open centroided mzML before loading into TidyMS/pyopenms for QC pipelines such
  as system conditioning or system suitability checks.
when_to_use_negative:
- Data are already in mzML or another open centroided format that TidyMS/pyopenms
  can read directly — conversion would be redundant.
- Analysis starts from a preprocessed feature table or extracted data matrix rather
  than raw vendor files, e.g. TidyMS data-curation filters (Rt > 90 s, RSD < 20%,
  D-ratio) on an existing matrix.
edam_operation: http://edamontology.org/operation_3257
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: ProteoWizard
  role: 'Vendor file converter: msconvert converts Waters RAW acquisitions to centroid-mode
    mzML (min distance = 0.01, snr = 1) for downstream TidyMS loading'
- name: TidyMS
  role: 'Downstream consumer: loads the converted mzML files with pyopenms and runs
    the system-conditioning / system-suitability QC pipelines'
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Reader backend inside TidyMS that parses the centroided mzML output of the
    conversion step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:1e1029b023b40753311da466d4d3839b5510dd9a091d0b3a5de8e8fe64ea98da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - ProteoWizard
  - Scipy
  - TidyMS
  - pyopenms
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3257
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# raw-ms-data-format-conversion

## Summary

Converts vendor-proprietary raw LC–MS files (e.g. Waters RAW) into centroid-mode mzML with ProteoWizard msconvert (min peak distance 0.01, SNR threshold 1) so TidyMS/pyopenms-based QC pipelines for untargeted metabolomics can load them for system-conditioning, system-suitability, and signal-drift assessment.

## When to use

Use when raw vendor LC–MS acquisition files (Waters RAW) must be converted to open centroided mzML before loading into TidyMS/pyopenms for QC pipelines such as system conditioning or system suitability checks.

## When NOT to use

- Data are already in mzML or another open centroided format that TidyMS/pyopenms can read directly — conversion would be redundant.
- Analysis starts from a preprocessed feature table or extracted data matrix rather than raw vendor files, e.g. TidyMS data-curation filters (Rt > 90 s, RSD < 20%, D-ratio) on an existing matrix.

## Inputs

- Vendor-proprietary raw LC–MS files (e.g. Waters .raw acquisitions from the MTBLS1919 deposit of NIST RM 8231 / SRM 1950 plasma runs)
- Batch/injection sequence metadata identifying SSS, pooled/intrastudy QC, blank, and study sample injections

## Outputs

- Centroid-mode mzML files (one per raw injection) readable by TidyMS via pyopenms
- Ordered set of mzML files ready for TidyMS system-conditioning and system-suitability pipelines

## How to apply

Locate the vendor-proprietary raw files — in the exemplar study these are the Waters RAW files deposited to MetaboLights under MTBLS1919 for NIST candidate RM 8231 and SRM 1950 plasma injections. Run ProteoWizard msconvert with centroid mode enabled, setting the minimum peak distance to 0.01 and a signal-to-noise threshold of 1, which yields the centroided mzML files TidyMS reads via pyopenms. Retain the batch/injection sequence (e.g. the 100-fold diluted system suitability solution injected before conditioning and at batch end) when organizing the converted files, since downstream TidyMS conditioning metrics depend on identifying SSS, pooled/intrastudy QC, and study sample injections in order. Point TidyMS at the converted mzML directory rather than the vendor files, because the pyopenms reader consumes mzML only. Validate conversion by confirming every injection in the batch produced an mzML and that SSS standards (Alogliptin, Phe-Phe, Leu-Enk) are detectable in extracted ion chromatograms from the converted data.

## Related tools

- **ProteoWizard** (Vendor file converter: msconvert converts Waters RAW acquisitions to centroid-mode mzML (min distance = 0.01, snr = 1) for downstream TidyMS loading)
- **TidyMS** (Downstream consumer: loads the converted mzML files with pyopenms and runs the system-conditioning / system-suitability QC pipelines) — https://github.com/griquelme/tidyms
- **pyopenms** (Reader backend inside TidyMS that parses the centroided mzML output of the conversion step)

## Examples

```
msconvert SSS_A.raw --mzML --filter "peakCasting" --mzML --filter "msLevel 1" ; # centroid min distance 0.01, snr 1
```

## Evaluation signals

- Every raw injection in the batch (e.g. all 31 Application-1 files, or the full MTBLS1919 set) yields exactly one mzML file with no conversion failures.
- Converted mzMLs are centroided (peak lists, not profile), matching the min distance 0.01 / snr 1 msconvert settings.
- Extracted ion chromatograms from converted data detect the SSS standards Alogliptin, Phe-Phe, and Leucine-Enkephalin at the expected retention windows.
- TidyMS pyopenms reader loads every converted file without errors and the conditioning pipeline runs to completion across all injections.
- Ordering of mzML files preserves the original acquisition sequence (SSS before conditioning, SSS at batch end) so downstream conditioning metrics map to the correct injections.

## Limitations

- Conversion depends on ProteoWizard support for the vendor format; profile-mode defaults must be overridden to centroid with the specified min-distance/SNR settings or downstream TidyMS feature detection behavior changes.
- The article only demonstrates the flow for Waters RAW files from NIST plasma reference material runs (MTBLS1919); other vendors are not validated here.
- TidyMS loading requires mzML via pyopenms; vendor-proprietary files cannot be read directly, so a failed or partial conversion blocks the whole QC pipeline.

## Evidence

- [other] Obtain the raw Waters RAW files from the MetaboLights MTBLS1919 deposit and convert them to centroid-mode mzML using ProteoWizard msconvert (min distance = 0.01, snr = 1).: [evidence span withheld — anchor status: not_found]
- [other] Load the mzML files with TidyMS (pyopenms reader) and map the batch sequence: [evidence span withheld — anchor status: not_found]
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository with the data set identifier MTBLS1919.: [evidence span withheld — anchor status: not_found]
- [results] The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [intro] Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms.: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms."
