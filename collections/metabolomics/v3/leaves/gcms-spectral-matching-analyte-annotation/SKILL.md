---
name: gcms-spectral-matching-analyte-annotation
description: Use when a processed GC-MS peak table (e.g. WiPP's merged high-quality
  peak set from mzML/mzData/NetCDF inputs) needs analyte identification by spectral
  matching, and identifications must be benchmarked against analytes confirmed in
  an original published study.
when_to_use_negative:
- The peak table already carries metabolite annotations or is already a feature/identifier
  table — re-running spectral matching adds nothing.
- No reference spectral library or published analyte list exists for the instrument/protocol,
  so matches cannot be validated.
- The WiPP classifiers have not been trained for the instrument and data acquisition
  method (peak annotation, ~1200 peaks per algorithm, is a prerequisite and is done
  only once per instrument/protocol).
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: WiPP
  role: Produces the merged high-quality peak table (via peak detection, output classification,
    and result integration) that is then annotated by spectral matching with the original
    study's settings
  repo: https://github.com/bihealth/WiPP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:374be24851e51a7b5a252fc858acaf1115cef30de1d63d3811c4f9cc1ff4ffe6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  - conda
  - git
  - libnetcdf11
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gcms-spectral-matching-analyte-annotation

## Summary

Annotates a merged high-quality GC-MS peak table (produced by WiPP's SVM-based peak picking) by spectral matching against a reference library/settings to identify analytes, then validates the identifications by comparing fold changes and p-values to a published analysis. On the MTBLS105 hepatocellular carcinoma plasma dataset this recovered 6 of 9 manually confirmed analytes.

## When to use

Use when a processed GC-MS peak table (e.g. WiPP's merged high-quality peak set from mzML/mzData/NetCDF inputs) needs analyte identification by spectral matching, and identifications must be benchmarked against analytes confirmed in an original published study.

## When NOT to use

- The peak table already carries metabolite annotations or is already a feature/identifier table — re-running spectral matching adds nothing.
- No reference spectral library or published analyte list exists for the instrument/protocol, so matches cannot be validated.
- The WiPP classifiers have not been trained for the instrument and data acquisition method (peak annotation, ~1200 peaks per algorithm, is a prerequisite and is done only once per instrument/protocol).

## Inputs

- Merged high-quality peak table from WiPP (classifier-labelled peaks across all peak detection algorithms)
- GC-MS raw data in mzML, mzData, or NetCDF format (e.g. 89 MTBLS105 SIM-mode plasma files organised in the Input_folder)
- Reference spectral matching settings / confirmed analyte list from the original published study

## Outputs

- Annotated high-quality peak table with analyte identifications from spectral matching
- Per-analyte fold changes and p-values comparable to the original study
- Quality flags per peak (high quality / shouldering / low quality)

## How to apply

First run the WiPP peak-picking step (`run_WiPP.sh pp`), which performs classifier hyperparameter optimisation, peak detection algorithm parameter optimisation, peak detection on the full dataset, output classification, and result integration, yielding one merged high-quality peak table. Then annotate this table by spectral matching using the same matching settings as the original reference study — for the MTBLS105 SIM-mode GC-qMS plasma data this means low-resolution settings (`high_resolution: False`, mass resolution below 1 Da). For each matched feature, compute fold changes and p-values across experimental conditions and compare them to the published values. Judge success by agreement with the manually confirmed analyte list: in the case study, 6 of 9 confirmed analytes were automatically identified, 3 were flagged as shouldering, and 100% of detected peaks were labelled high quality.

## Related tools

- **WiPP** (Produces the merged high-quality peak table (via peak detection, output classification, and result integration) that is then annotated by spectral matching with the original study's settings) — https://github.com/bihealth/WiPP

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- The number of automatically identified analytes matches the reference study's confirmed list (6 of 9 confirmed analytes recovered on MTBLS105).
- Fold changes and p-values of identified analytes are comparable to those reported in the original published analysis.
- 100% of peaks in the final table carry a high-quality label from the SVM classifier.
- Shouldering peaks (3 of the 9 analytes in the case study) are explicitly flagged rather than silently dropped.
- The Input_folder/config.yaml setup (SIM-mode `high_resolution: False`, alkane list e.g. c10,...,c34, Training/Optimization/Wash subdirectories) matches the acquisition method before annotation.

## Limitations

- Not all confirmed analytes are recovered automatically: in the MTBLS105 case study only 6 of 9 were identified and 3 were flagged as shouldering.
- Trained SVM classifiers are specific to the instrument and data acquisition method; the example classifier provided with WiPP is for test data only and must not be reused on real projects.
- Validation depends on reproducing the original study's spectral matching settings; mismatched settings will change which analytes are called identified.
- WiPP is not supported on Ubuntu 18 (Bionic Beaver) due to lacking support of incorporated R packages; tested on Ubuntu 16 and CentOS 7.6.

## Evidence

- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set for immediate: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set for immediate"
- [readme] classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
- [readme] Once trained, the SVM classifiers can be reused on other datasets generated with the same instrument and data acquisition method.: "Once trained, the SVM classifiers can be reused on other datasets generated with the same instrument and data acquisition method."
