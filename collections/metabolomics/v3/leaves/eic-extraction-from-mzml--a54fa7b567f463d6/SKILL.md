---
name: eic-extraction-from-mzml
description: Use when raw mzML MS1 spectra and a list of detected features (m/z, rt)
  must be converted into aligned EIC data matrices — e.g. to train or apply PeakDetective
  peak-artifact classifiers.
when_to_use_negative:
- Input is already a feature-intensity table with no raw mzML access — EIC extraction
  requires raw scans.
- Data is MS2 or fragmentation-centric and no MS1 peak curation is intended; PeakDetective
  operates on MS1 EICs.
- No peak list is available and you intend de novo detection rather than curation
  of pre-detected features.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: PeakDetective
  role: Provides rawData.readRawDataFile for parsing mzML MS1 spectra with a 25.5
    ppm tolerance and PeakDetective.makeDataMatrix for generating the aligned EIC
    data matrix used by the smoother and active-learning classifier.
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Upstream peak detector whose mz/rt peak list (peaks_formatted.csv) supplies
    the features around which EICs are extracted.
- name: pyteomics
  role: Required package dependency used by PeakDetective for mzML parsing support
    during raw data reading.
- name: numpy
  role: Required numerical package used by PeakDetective for array handling in EIC
    data matrix construction.
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:0dea92221e9ec7dc73642a683ac0c27fe0b971df2739290624e18b147e6ad165
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - PeakDetective
  - Python
  - XCMS
  - numpy
  - pyteomics
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# eic-extraction-from-mzml

## Summary

Extract extracted-ion chromatograms (EICs) of metabolomic features from raw mzML MS1 files as fixed-resolution data matrices so that peak curation models (e.g. PeakDetective's autoencoder/classifier) can operate on reproducible peak representations.

## When to use

Use when raw mzML MS1 spectra and a list of detected features (m/z, rt) must be converted into aligned EIC data matrices — e.g. to train or apply PeakDetective peak-artifact classifiers.

## When NOT to use

- Input is already a feature-intensity table with no raw mzML access — EIC extraction requires raw scans.
- Data is MS2 or fragmentation-centric and no MS1 peak curation is intended; PeakDetective operates on MS1 EICs.
- No peak list is available and you intend de novo detection rather than curation of pre-detected features.

## Inputs

- Raw mzML files (MS1 spectra) in a single folder
- Peak list CSV with 'mz' and 'rt' columns (e.g. XCMS peaks_formatted.csv)
- ms1ppm tolerance setting (e.g. 25.5 ppm)
- align flag (True = perform RT alignment, False = do not align)

## Outputs

- EIC data matrix X with one fixed-resolution chromatogram per feature per sample (resolution=60 points per minute, window=1.0 min)
- Raw data objects (PeakDetective.rawData) holding parsed MS1 scans per mzML file

## How to apply

Initialize a PeakDetective.PeakDetective object with numCores and resolution=60 (number of data points per 1-min EIC), then read each mzML file into a PeakDetective.rawData object via readRawDataFile with an ms1ppm tolerance of 25.5 ppm. Load the peak list (e.g. peaks_formatted.csv) as an mz/rt table with pandas, and call makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=align) to build the EIC matrix, where align=True performs retention-time alignment across samples. The window parameter sets the EIC window size in minutes (1.0 min by default), and the resulting matrix X is the input required for both trainSmoother and trainClassifierActive. Use RT alignment only when run order or drift justifies it, since alignment changes peak shapes used downstream.

## Related tools

- **PeakDetective** (Provides rawData.readRawDataFile for parsing mzML MS1 spectra with a 25.5 ppm tolerance and PeakDetective.makeDataMatrix for generating the aligned EIC data matrix used by the smoother and active-learning classifier.) — https://github.com/pattilab/PeakDetective
- **XCMS** (Upstream peak detector whose mz/rt peak list (peaks_formatted.csv) supplies the features around which EICs are extracted.)
- **pyteomics** (Required package dependency used by PeakDetective for mzML parsing support during raw data reading.)
- **numpy** (Required numerical package used by PeakDetective for array handling in EIC data matrix construction.)

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True)
```

## Evaluation signals

- Data matrix X has one row per feature and one EIC per sample, with each EIC at resolution=60 data points per 1-min window of 1.0 min.
- EIC extraction succeeds only after every mzML file in mzmlFolder has been read with readRawDataFile using ms1ppm=25.5.
- trainSmoother and trainClassifierActive receive the same matrix X produced by makeDataMatrix — if either errors, the EIC matrix was malformed.
- Peak list must contain 'mz' and 'rt' columns before makeDataMatrix is called (loaded via pd.read_csv(peakFile)[['mz','rt']]).

## Limitations

- model parameters such as resolution=60 and window=1.0 are documented as 'not recommended to change' — altering them affects the latent representation the classifier was trained on.
- The align flag changes peak shapes via RT alignment; choosing align=True when drift is absent can distort EICs unnecessarily.
- EIC quality depends on the ms1ppm tolerance; the README example uses 25.5 ppm, but different instruments may require tuning.

## Evidence

- [results] Task card 004 workflow with ms1ppm tolerance: [evidence span withheld — anchor status: not_found]
- [results] Task card 005 workflow with ppm tolerance: [evidence span withheld — anchor status: not_found]
