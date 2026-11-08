---
name: metabolomics-eic-extraction
description: Use when raw LC/MS mzML files and a detected peak list (m/z, RT columns,
  e.g. from XCMS) must be converted into aligned EIC matrices before peak classification
  or curation.
when_to_use_negative:
- Input is already a curated/quantified feature table (peak intensities per sample)
  rather than raw mzML files — no EIC extraction is needed
- Peak list has not been generated yet — run XCMS first (detection_helper.PeakList().runXCMS
  with peakWidth=(13.8,114.6) s, s2n=13.6, prefilter=5) to produce the peak list
- Data are not LC/MS metabolomics profiles (e.g. proteomics MS/MS spectra) — the mzML
  MS1 EIC assumption does not apply
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakDetective
  role: Provides rawData() reader (readRawDataFile with ppm tolerance) and makeDataMatrix()
    to build aligned EIC matrices from mzML + peak list
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Generates the upstream peak list (m/z, RT) from the mzML files that defines
    which EICs are extracted
- name: fastdtw
  role: Dependency used for RT alignment of chromatograms before EIC extraction (align=True)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:9278253dc96a1e3d94752502bb1a34fb57c1a761bf93d9868fbcc8006c613a52
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pattilab/PeakDetective
  tools:
  - NeatMS
  - PeakDetective
  - Python
  - TensorFlow
  - XCMS
  - fastdtw
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# metabolomics-eic-extraction

## Summary

Build EIC (extracted-ion chromatogram) data matrices for LC/MS peaks from raw mzML files and an XCMS-generated peak list, producing the per-peak EIC representations that PeakDetective's smoother and classifier require for downstream peak curation.

## When to use

Use when raw LC/MS mzML files and a detected peak list (m/z, RT columns, e.g. from XCMS) must be converted into aligned EIC matrices before peak classification or curation.

## When NOT to use

- Input is already a curated/quantified feature table (peak intensities per sample) rather than raw mzML files — no EIC extraction is needed
- Peak list has not been generated yet — run XCMS first (detection_helper.PeakList().runXCMS with peakWidth=(13.8,114.6) s, s2n=13.6, prefilter=5) to produce the peak list
- Data are not LC/MS metabolomics profiles (e.g. proteomics MS/MS spectra) — the mzML MS1 EIC assumption does not apply

## Inputs

- Raw LC/MS data files in mzML format (folder of .mzML files)
- XCMS-generated peak list CSV with 'mz' and 'rt' columns (e.g. peaks_formatted.csv)
- ms1ppm mass tolerance parameter (25.5 ppm in the reference workflow)
- align flag controlling RT alignment (True/False)

## Outputs

- EIC data matrix X mapping each peak (m/z, RT) to its extracted-ion chromatogram across samples
- Populated PeakDetective object holding raw_data (parsed mzML) ready for model training

## How to apply

Load each mzML file into a PeakDetective.rawData() object with readRawDataFile(mzmlFolder + '/' + file, ms1ppm) using ms1ppm = 25.5 ppm tolerance, and read the XCMS peak list keeping only the 'mz' and 'rt' columns. Initialize a PeakDetective object with numCores and resolution, then call makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=align) with resolution = 60 data points per 1-min EIC and window = 1.0 min. Set align = True so retention-time alignment (via fastdtw) is performed before extraction, ensuring peaks across samples are comparable. The returned matrix X of EICs is the input required for trainSmoother and trainClassifierActive; without RT alignment or the correct ppm tolerance, EICs for the same metabolite will be misaligned across samples and degrade classifier training.

## Related tools

- **PeakDetective** (Provides rawData() reader (readRawDataFile with ppm tolerance) and makeDataMatrix() to build aligned EIC matrices from mzML + peak list) — https://github.com/pattilab/PeakDetective
- **XCMS** (Generates the upstream peak list (m/z, RT) from the mzML files that defines which EICs are extracted)
- **fastdtw** (Dependency used for RT alignment of chromatograms before EIC extraction (align=True))

## Examples

```
integ = PeakDetective.PeakDetective(numCores=10, resolution=60); X = integ.makeDataMatrix(raw_data, peaklist['mz'].values, peaklist['rt'].values, align=True)
```

## Evaluation signals

- makeDataMatrix returns a matrix X whose row count matches the number of peaks in the peak list (mz/rt pairs)
- EICs are built with resolution = 60 points per 1-min window and window = 1.0 min, so each EIC has a consistent sampling density
- align=True produces RT-aligned EICs; rerunning with align=False should yield visibly misaligned chromatograms across samples
- Parsed rawData objects exist for every .mzML file in the folder (one PeakDetective.rawData() per file)
- Downstream trainSmoother accepts the matrix without shape errors, indicating valid EIC dimensions

## Limitations

- EIC quality depends on the ms1ppm tolerance (25.5 ppm in this workflow); a mismatched ppm tolerance relative to the instrument's mass accuracy will fragment EICs
- RT alignment (align=True) adds computation time via fastdtw; skipping it degrades cross-sample peak correspondence
- Parameters are described in the README as 'model parameters, not recomended to change' — resolution and window should be kept at 60 and 1.0 unless the LC method demands otherwise

## Evidence
