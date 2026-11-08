---
name: raw-lcms-peak-picking
description: Use when starting from raw LC-MS/MS files of an untargeted metabolomics
  study (e.g. Metabolomics WorkBench ST002044) and you need unique MS/MS spectra in
  NIST .msp format per ESI+/ESI− mode as input for library search or fingerprint prediction.
when_to_use_negative:
- Input is already a deconvoluted .msp spectral file — go directly to library search
  or IDSL_MINT fingerprint prediction instead
- The goal is MS1-only quantification or peak-area statistics with no MS/MS scans
  to deconvolute
- The raw data are not LC-MS/MS metabolomics runs with fragment spectra (e.g. GC-MS
  EI data or direct infusion without chromatography)
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: IDSL.IPA
  role: Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and
    their associated MS/MS scans prior to deconvolution
- name: IDSL.CSA
  role: Deconvolutes MS/MS spectra by associating fragment spectra with precursor
    features, per ionization mode, and deduplicates to unique .msp spectra; also used
    for downstream library matching
- name: IDSL_MINT
  role: 'Downstream consumer of the extracted .msp files: requires MSP blocks with
    PrecursorMZ row entries and parses SMILES/InChI labels for fingerprint training
    and prediction'
  repo: https://github.com/idslme/IDSL_MINT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:bf9ba44dfc96ff744d6cf179fdecb5f88288a110ed976baead499359cb4039ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL.IPA
  - IDSL_MINT
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# raw-lcms-peak-picking

## Summary

Extracts deconvoluted, deduplicated MS/MS spectra from raw LC-MS/MS data of untargeted metabolomics studies using IDSL.IPA chromatographic peak-picking followed by IDSL.CSA spectral deconvolution, exporting unique spectra per ionization mode in NIST .msp format for downstream annotation or deep learning models.

## When to use

Use when starting from raw LC-MS/MS files of an untargeted metabolomics study (e.g. Metabolomics WorkBench ST002044) and you need unique MS/MS spectra in NIST .msp format per ESI+/ESI− mode as input for library search or fingerprint prediction.

## When NOT to use

- Input is already a deconvoluted .msp spectral file — go directly to library search or IDSL_MINT fingerprint prediction instead
- The goal is MS1-only quantification or peak-area statistics with no MS/MS scans to deconvolute
- The raw data are not LC-MS/MS metabolomics runs with fragment spectra (e.g. GC-MS EI data or direct infusion without chromatography)

## Inputs

- Raw LC-MS/MS data files from an untargeted metabolomics study (e.g. Metabolomics WorkBench study ST002044)
- Ionization mode configuration (ESI+ and ESI− processed separately)
- IDSL.IPA parameter set for MS1 feature detection and MS/MS scan association

## Outputs

- Unique MS/MS spectra in NIST .msp format for ESI+ mode (e.g. 3,386 spectra for ST002044)
- Unique MS/MS spectra in NIST .msp format for ESI− mode (e.g. 1,901 spectra for ST002044)
- MSP blocks containing PrecursorMZ (and optionally SMILES/InChI, Precursor_type, Collision_energy) row entries

## How to apply

Download the raw LC-MS/MS data for the study and run IDSL.IPA to perform chromatographic peak-picking, detecting MS1 features and their associated MS/MS scans. Then run the IDSL.CSA R package to deconvolute the MS/MS spectra, associating fragment spectra with precursor features, processing ESI+ and ESI− modes separately. Deduplicate deconvolved spectra within each ionization mode and export the unique MS/MS spectra as NIST .msp text blocks; every MSP block must include a PrecursorMZ row entry, and SMILES/InChI entries in blocks can be canonicalized with RDKit when needed. Validate by counting spectrum blocks per mode — for ST002044 the expected yield is 3,386 (ESI+) and 1,901 (ESI−). These .msp files then serve as the model evaluation or annotation set, where library match coverage can be quantified (e.g. IDSL.CSA annotated 638/3386 positive and 299/1901 negative spectra against NIST 20 and LipidBlast with spectral entropy score ≥ 0.75).

## Related tools

- **IDSL.IPA** (Chromatographic peak-picking on raw LC-MS/MS data to detect MS1 features and their associated MS/MS scans prior to deconvolution)
- **IDSL.CSA** (Deconvolutes MS/MS spectra by associating fragment spectra with precursor features, per ionization mode, and deduplicates to unique .msp spectra; also used for downstream library matching)
- **IDSL_MINT** (Downstream consumer of the extracted .msp files: requires MSP blocks with PrecursorMZ row entries and parses SMILES/InChI labels for fingerprint training and prediction) — https://github.com/idslme/IDSL_MINT

## Evaluation signals

- Count unique MS/MS spectrum blocks in each output .msp file; for study ST002044 the ESI+ file must contain 3,386 spectra and the ESI− file 1,901, matching the reported values
- Every MSP block must include a PrecursorMZ row entry — blocks missing this field are invalid
- Deconvolution must correctly associate each fragment spectrum with a precursor MS1 feature (no orphan MS/MS scans)
- Spectra are deduplicated within each ionization mode; duplicate spectrum blocks indicate the deduplication step failed
- Downstream library matching coverage is a sanity check: IDSL.CSA annotated 638/3386 (ESI+) and 299/1901 (ESI−) against NIST 20 and LipidBlast using spectral entropy score ≥ 0.75

## Limitations

- Spectrum counts are study- and instrument-dependent; the 3,386/1,901 values hold only for ST002044 and must be re-derived for other datasets
- A large proportion (>80%) of extracted MS/MS spectra may have no hit in mass spectral libraries, so extraction alone does not guarantee annotation
- MSP parsing is header-sensitive: when multiple similar headers appear in an MSP block, the longest content is selected, and comments are read from 'comment:'/'comments:' entries

## Evidence

- [results] IDSL.CSA extracted 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for study ST002044: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] Raw data are processed to NIST .msp format as the model evaluation set: "First, we processed the raw data for this study to extract MS/MS spectra in the NIST.msp format that can be used as the model evaluation set."
- [readme] MSP blocks require PrecursorMZ entries: "MSP blocks must include `PrecursorMZ: ` row entries."
