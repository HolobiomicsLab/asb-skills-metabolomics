---
name: lc-ms-feature-detection
description: Use when raw UHPLC-HRMS2 DDA data (Thermo RAW converted to mzXML/mzML)
  must be converted into a per-sample feature quantification table plus per-feature
  MS2 spectra (.mgf), before structural annotation or FBMN.
when_to_use_negative:
- Input is already a feature quantification table with aligned features — the ENPKG
  approach deliberately skips classical RT/m/z alignment across samples
- Data are MS1-only without MS2 scans — the feature filter keeps only features with
  an associated MS2 scan, so the pipeline would retain nothing
- Peak detection for proteomics-style workflows requiring different noise/deconvolution
  settings — the thresholds here (1.0E4 MS1 noise, 5.0E5 minimum height) are tuned
  for plant-extract UHPLC-HRMS2 metabolomics
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Primary tool performing mass detection, ADAP chromatogram building, wavelet
    deconvolution, isotope grouping, feature filtering, and 'Export to GNPS' for each
    sample
  repo: https://github.com/enpkg/enpkg_workflow
- name: ProteoWizard
  role: Converts Thermo RAW LC-MS files to mzXML/mzML format as the entry point for
    MZmine processing
- name: matchms
  role: Downstream consumer of the exported .mgf MS2 spectra for matchms-based ISDB
    spectral matching
  repo: https://github.com/enpkg/enpkg_workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:257d7ea9c1c4e2fa7384f200b4189d741c14893b077ad8a48d4cbe443afd315d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - MZmine
  - ProteoWizard
  - matchms
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# LC-MS Feature Detection

## Summary

Per-sample feature detection and MS2 spectra export from LC-HRMS2 data using MZmine (mass detection, ADAP chromatogram building, wavelet deconvolution, isotope grouping, filtering), producing a feature quantification table and .mgf MS2 spectra for downstream annotation and molecular networking in the ENPKG framework.

## When to use

Use when raw UHPLC-HRMS2 DDA data (Thermo RAW converted to mzXML/mzML) must be converted into a per-sample feature quantification table plus per-feature MS2 spectra (.mgf), before structural annotation or FBMN.

## When NOT to use

- Input is already a feature quantification table with aligned features — the ENPKG approach deliberately skips classical RT/m/z alignment across samples
- Data are MS1-only without MS2 scans — the feature filter keeps only features with an associated MS2 scan, so the pipeline would retain nothing
- Peak detection for proteomics-style workflows requiring different noise/deconvolution settings — the thresholds here (1.0E4 MS1 noise, 5.0E5 minimum height) are tuned for plant-extract UHPLC-HRMS2 metabolomics

## Inputs

- Thermo RAW LC-MS files converted to mzXML (via MSConvert/ProteoWizard) or mzML
- Per-sample LC-HRMS2 DDA data in positive and/or negative ionization modes

## Outputs

- Feature quantification table (m/z, RT, area/intensity per feature) for each sample
- Per-feature MS2 spectra exported as .mgf files
- Raw LC-HRMS2 data exported as .mzML files

## How to apply

Process each sample individually rather than relying on dataset-wide RT-based alignment, which the authors note hinders comparison across batches. In MZmine, first detect masses with the centroid mass detector (noise level 1.0E4 for MS1, 0 for MS2), then build chromatograms with the ADAP chromatogram builder (minimum group size 5 scans, group intensity threshold 1.0E4, highest intensity 5.0E5, m/z tolerance 12 ppm). Deconvolute peaks with the wavelets (ADAP) algorithm using the intensity window S/N estimator (S/N 10, minimum feature height 5.0E5, coefficient area threshold 50, peak duration 0.02–0.5 min, RT wavelet range 0.01–0.03 min). Group isotopes with the isotope peaks grouper (12 ppm, 0.01 min absolute RT tolerance, max charge 2, most intense isotope as representative), then filter each feature list to keep only features with an associated MS2 scan and RT between 0.5 and 8.0 min. Finally export the deconvoluted feature quantification table and per-feature MS2 spectra via the 'Export to GNPS' module so downstream matchms-based ISDB annotation and molecular networking can consume them.

## Related tools

- **MZmine** (Primary tool performing mass detection, ADAP chromatogram building, wavelet deconvolution, isotope grouping, feature filtering, and 'Export to GNPS' for each sample) — https://github.com/enpkg/enpkg_workflow
- **ProteoWizard** (Converts Thermo RAW LC-MS files to mzXML/mzML format as the entry point for MZmine processing)
- **matchms** (Downstream consumer of the exported .mgf MS2 spectra for matchms-based ISDB spectral matching) — https://github.com/enpkg/enpkg_workflow

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && cd enpkg_full && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Feature quantification table exists with expected columns (m/z, RT, area/intensity per feature)
- Exported .mgf files contain an MS2 spectrum for each retained feature (features kept must have an associated MS2 scan)
- All retained features have retention times within the 0.5–8.0 min window
- Per-sample feature counts are consistent with the paper's reported scale (~788,623 features in PI and ~364,967 in NI before per-sample filtering)
- Isotope grouping applied with max charge 2 and the most intense isotope chosen as the representative peak

## Limitations

- Classical RT/mz-based feature alignment across many samples is problematic — batch effects and recomputation on new samples motivate the per-sample processing design of this step
- The filtering step discards features lacking MS2 scans or with RT outside 0.5–8.0 min, so features outside this window are not exported for annotation
- Parameters (12 ppm, 5.0E5 intensity cutoffs, 0.02–0.5 min peak duration) are specific to the UHPLC-HRMS2 plant extract setup and may not transfer to other instrumental configurations

## Evidence

- [methods] Centroid mass detection with MS1 noise threshold 1.0E4: "the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1"
- [methods] ADAP chromatogram builder parameters: "The ADAP chromatogram builder was used and set to a minimum group size of scans of 5, minimum group intensity threshold of 1.0E4, minimum highest intensity of 5.0E5"
- [methods] Feature filtering: MS2 scan required, RT 0.5–8.0 min: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [results] Feature detection step in the ENPKG workflow uses MZmine: "feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine"
- [results] Reported feature counts in PI and NI modes: "788,623 and 364,967 features in positive ionization (PI) and negative ionization (NI) modes, respectively"
- [readme] Per-sample individual processing is a core design choice: [evidence span withheld — anchor status: external_doc]
