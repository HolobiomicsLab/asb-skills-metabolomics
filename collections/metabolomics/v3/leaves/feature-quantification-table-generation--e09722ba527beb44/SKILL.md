---
name: feature-quantification-table-generation
description: Use when raw LC-HRMS2 DDA metabolomics data (Thermo RAW / mzML / mzXML)
  must be converted, per sample, into a feature quantification table (m/z, RT, area/intensity)
  plus MS2 .mgf files for FBMN or ISDB annotation.
when_to_use_negative:
- Input is already an aligned N x M feature quantification table — the article's motivation
  is precisely that RT-based feature alignment across batches is problematic, so re-running
  feature detection is redundant
- The goal is downstream KG querying (SPARQL over the RDF knowledge graph) or structural
  annotation rather than feature detection and export
- Data are not LC-MS/MS DDA metabolomics acquisitions processed with MZmine-compatible
  peak-picking parameters
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: 'Performs feature detection: mass detection, ADAP chromatogram building, wavelets
    (ADAP) peak deconvolution, isotope grouping, feature filtering, and ''Export to
    GNPS'' module for the quantification table and .mgf files'
  repo: http://mzmine.github.io/
- name: ProteoWizard
  role: MSConvert converts Thermo RAW LC-MS files to mzXML format as the entry point
    for MZmine processing
- name: matchms
  role: Downstream consumer of the exported .mgf MS2 spectra for matchms-based ISDB
    spectral matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:3fc0c57947406abe6dac0b9d2ec2e5212710c51701e65a6468310782748231bb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine
  - ProteoWizard
  - matchms
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Feature Quantification Table Generation

## Summary

Per-sample LC-HRMS2 feature detection and export using MZmine within the ENPKG workflow: deconvolute chromatographic peaks, filter features, and export a feature quantification table plus per-feature MS2 spectra (.mgf) that downstream FBMN and ISDB annotation steps consume.

## When to use

Use when raw LC-HRMS2 DDA metabolomics data (Thermo RAW / mzML / mzXML) must be converted, per sample, into a feature quantification table (m/z, RT, area/intensity) plus MS2 .mgf files for FBMN or ISDB annotation.

## When NOT to use

- Input is already an aligned N x M feature quantification table — the article's motivation is precisely that RT-based feature alignment across batches is problematic, so re-running feature detection is redundant
- The goal is downstream KG querying (SPARQL over the RDF knowledge graph) or structural annotation rather than feature detection and export
- Data are not LC-MS/MS DDA metabolomics acquisitions processed with MZmine-compatible peak-picking parameters

## Inputs

- Thermo RAW LC-MS/MS DDA files per sample (positive and/or negative ionization modes)
- mzXML files converted from RAW via MSConvert (ProteoWizard)
- mzML raw LC-HRMS2 data

## Outputs

- Deconvoluted feature quantification table with m/z, RT, and area/intensity per feature
- Per-feature MS2 spectra exported as .mgf files
- Filtered feature lists (MS2-associated features, RT 0.5–8.0 min)

## How to apply

Convert Thermo RAW files to mzXML with MSConvert (ProteoWizard) as the MZmine entry point, then process each sample individually (per ENPKG's sample-centric design, which avoids cross-batch RT alignment). In MZmine, detect masses with the centroid mass detector (noise level 1.0E4 for MS level 1, 0 for MS level 2); build chromatograms with the ADAP chromatogram builder (minimum group size 5 scans, minimum group intensity threshold 1.0E4, minimum highest intensity 5.0E5, m/z tolerance 12 ppm); deconvolute peaks with the wavelets (ADAP) algorithm using the intensity window S/N estimator (S/N 10, minimum feature height 5.0E5, coefficient area threshold 50, peak duration 0.02–0.5 min, RT wavelet range 0.01–0.03 min); group isotopes with the isotope peaks grouper (12 ppm m/z tolerance, 0.01 min absolute RT tolerance, max charge 2, most intense isotope as representative). Filter each feature list with the feature filtering module to keep only features with an associated MS2 scan and RT between 0.5 and 8.0 min, then export the deconvoluted feature quantification table and MS2 spectra as .mgf using the 'Export to GNPS' module so matchms-based ISDB annotation and molecular networking can consume them.

## Related tools

- **MZmine** (Performs feature detection: mass detection, ADAP chromatogram building, wavelets (ADAP) peak deconvolution, isotope grouping, feature filtering, and 'Export to GNPS' module for the quantification table and .mgf files) — http://mzmine.github.io/
- **ProteoWizard** (MSConvert converts Thermo RAW LC-MS files to mzXML format as the entry point for MZmine processing)
- **matchms** (Downstream consumer of the exported .mgf MS2 spectra for matchms-based ISDB spectral matching)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git; cd enpkg_full; uv sync; sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Feature quantification table exists with expected columns (m/z, RT, area/intensity per feature)
- Exported .mgf files contain MS2 spectra for each retained feature
- Every retained feature has an associated MS2 scan and RT within 0.5–8.0 min
- Feature counts are in the expected order of magnitude relative to the reference run (788,623 features in PI and 364,967 in NI for the full data set before per-sample filtering)
- Deconvolution parameters applied match the reported settings (S/N 10, minimum feature height 5.0E5, peak duration 0.02–0.5 min, 12 ppm m/z tolerance)

## Limitations

- The noise (1.0E4) and minimum feature height (5.0E5) thresholds are instrument- and dataset-specific; applying them unchanged to other acquisitions may over- or under-detect features
- The per-sample design exists because feature alignment based on RT hinders comparison of samples from different batches or analyzed on different instruments; this skill deliberately does not produce cross-sample alignment
- The surrounding ENPKG pipeline assumes inputs formatted exactly like the reference datasets (e.g. the Zenodo example dataset); the emikg README warns it 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset'

## Evidence

- [methods] the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1: "the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1"
- [methods] The ADAP chromatogram builder was used and set to a minimum group size of scans of 5, minimum group intensity threshold of 1.0E4, minimum highest intensity of 5.0E5: "The ADAP chromatogram builder was used and set to a minimum group size of scans of 5, minimum group intensity threshold of 1.0E4, minimum highest intensity of 5.0E5"
- [methods] Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min.: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [results] 788,623 and 364,967 features in positive ionization (PI) and negative ionization (NI) modes, respectively: "788,623 and 364,967 features in positive ionization (PI) and negative ionization (NI) modes, respectively"
- [readme] To allow for iterative addition of samples over time, data from each sample is processed individually.: [evidence span withheld — anchor status: external_doc]
