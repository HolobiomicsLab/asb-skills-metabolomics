---
name: lcms-peak-curation
description: Use when you have raw mzML files plus an XCMS-generated peak list (m/z
  and rt columns) and need to remove artifact peaks — e.g. replicate, blank, or spiked-standard
  metabolomics data where peak quality determines downstream statistics.
when_to_use_negative:
- Input is already a curated feature table with no accompanying raw mzML files — curatePeaks
  needs raw EICs to score peak shapes
- The goal is peak detection from raw profiles rather than curation of an existing
  XCMS-detected peak list
- You cannot label even a small number of peaks and no pre-trained weights for a matching
  LC/MS method/sample type exist
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakDetective
  role: Semisupervised deep learning package that reads mzML raw data, generates EICs,
    and classifies XCMS-detected peaks as artifacts or true peaks via curatePeaks
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: 'R tool used upstream to detect the peak list (CentWave parameters: peakWidth=(13.8,114.6)
    s, s2n=13.6, prefilter=5) that PeakDetective then curates'
- name: pyteomics
  role: Required Python package used by PeakDetective for reading raw mzML data
- name: numpy
  role: Required Python dependency for array handling in the curation pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:1d3934b413e07ea7ea72bb6240db84f09d1ea2e4364c6e6225f4de54a6a00394
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
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# lcms-peak-curation

## Summary

Classifies LC/MS-detected peaks as artifacts or true peaks using PeakDetective, a semisupervised deep learning pipeline in which an autoencoder learns a latent peak representation and an actively-learned classifier scores each peak, curation being applied at a threshold such as 0.8.

## When to use

Use when you have raw mzML files plus an XCMS-generated peak list (m/z and rt columns) and need to remove artifact peaks — e.g. replicate, blank, or spiked-standard metabolomics data where peak quality determines downstream statistics.

## When NOT to use

- Input is already a curated feature table with no accompanying raw mzML files — curatePeaks needs raw EICs to score peak shapes
- The goal is peak detection from raw profiles rather than curation of an existing XCMS-detected peak list
- You cannot label even a small number of peaks and no pre-trained weights for a matching LC/MS method/sample type exist

## Inputs

- Raw LC/MS data in mzML format (e.g. mzmlFolder of .mzML files)
- XCMS-generated peak list CSV with mz and rt columns (e.g. peaks_formatted.csv / peaks.csv)
- Trained PeakDetective model weights (PeakDetectiveObject directory) OR interactive active-learning labels
- Classification cutoff (e.g. 0.8) and ms1ppm tolerance (e.g. 25.5)

## Outputs

- peaks_curated.csv (peak list classified as artifact or true peak)
- peak_scores.csv (per-peak classifier scores)
- peak_intensities.csv (curated peak intensities)
- Bimodal score histogram with TPR/FDR estimates per cutoff during training

## How to apply

Load raw mzML files with rawData.readRawDataFile using an ms1ppm tolerance of 25.5, read the XCMS peak list (peaks_formatted.csv) extracting only the mz and rt columns, and initialize a PeakDetective object. Generate EICs with makeDataMatrix (align=True for RT alignment), then either train the smoother and classifier via active learning (<100 user-labeled peaks over 5-10 iterations until scores are bimodal) or load saved model weights from a PeakDetectiveObject directory. Run integ.curatePeaks(raw_data, peaklist, threshold=cutoff, align=True) with cutoff=0.8 to score and classify every peak, writing peaks_curated.csv, peak_scores.csv, and peak_intensities.csv. The rationale is that the autoencoder's latent representation plus a semisupervised classifier captures peak-shape artifacts that simple heuristics miss, and the classifier can be tailored to a specific LC/MS method and sample type.

## Related tools

- **PeakDetective** (Semisupervised deep learning package that reads mzML raw data, generates EICs, and classifies XCMS-detected peaks as artifacts or true peaks via curatePeaks) — https://github.com/pattilab/PeakDetective
- **XCMS** (R tool used upstream to detect the peak list (CentWave parameters: peakWidth=(13.8,114.6) s, s2n=13.6, prefilter=5) that PeakDetective then curates)
- **pyteomics** (Required Python package used by PeakDetective for reading raw mzML data)
- **numpy** (Required Python dependency for array handling in the curation pipeline)

## Examples

```
integ.load('example_data_for_colab/xcms_example_data/PeakDetectiveObject/'); peak_curated,peak_scores,peak_intensities = integ.curatePeaks(raw_data,peaklist,threshold=0.8,align=True)
```

## Evaluation signals

- On the P. pastoris spiked-standard dataset (81 metabolite standards), the fraction of all XCMS-detected peaks classified as true peaks should be ~66.7 +/- 0.34% and the fraction of spiked-metabolite features ~83.9 +/- 0.81%, with the difference significant at p<0.0001
- During active learning, after 5-10 labeling iterations the score histogram should become bimodal and val_mean_absolute_error should drop dramatically
- Trained model achieves usable TPR/FDR trade-offs on <100 user-labeled validation/training peaks
- Output files peaks_curated.csv, peak_scores.csv, and peak_intensities.csv are produced with one entry per input peak
- Classifier accuracy reported separately on training and validation peaks each active-learning round

## Limitations

- PeakDetective must be tailored to a specific LC/MS method and sample type to maximize performance; weights trained on one dataset may not transfer
- The active-learning classifier requires interactive user labeling (numValPeaks=10, 10 peaks per round) and ≥5-10 iterations before the score distribution is bimodal
- Running XCMS or mz.unity degeneracy removal requires R (>v4.0) installed and R package installation takes >20 min
- Model parameters (resolution=60, window=1.0 min) are marked as not recommended to change

## Evidence
