---
name: metabolomics-feature-scoring
description: Use when you have an XCMS-generated peak list (mz/rt columns) from LC/MS
  metabolomics data plus raw mzML files, and need to separate true peaks from artifacts
  like noise or co-eluting degeneracies before downstream statistics.
when_to_use_negative:
- When you have only a feature intensity table with no raw mzML data — PeakDetective
  extracts low-dimensional latent representations from raw extracted ion chromatograms
  and cannot score peaks without the underlying chromatographic scans
- When the peak list has not been generated yet and you need peak detection itself
  rather than curation — this skill assumes an existing XCMS peak list as input, though
  PeakDetective can also run XCMS via detection_helper.PeakList().runXCMS
- When you need fully automated classification with zero user input and cannot afford
  the 5–10 active learning labeling rounds (or loading a pretrained model) — PeakDetective
  is semisupervised and requires labeled peaks to train
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PeakDetective
  role: 'Primary scoring/curation engine: trains an autoencoder latent representation
    and an active-learning classifier over EIC data matrices, then applies curatePeaks
    to score each XCMS-detected peak as artifact or true peak'
  repo: https://github.com/pattilab/PeakDetective
- name: XCMS
  role: Upstream peak detection tool whose peak list (peaks_formatted.csv) serves
    as the input that PeakDetective curates; can be invoked via detection_helper.PeakList().runXCMS
- name: pyteomics
  role: Required dependency used by PeakDetective for reading raw mzML files into
    rawData objects
- name: numpy
  role: Required dependency for numerical operations on EIC data matrices and peak
    score arrays
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00764
    title: 'PeakDetective: A Semisupervised Deep Learning-Based Approach for Peak
      Curation in Untargeted Metabolomics'
  source_content_hash: sha256:a45006f792754315f20992ad9bb71a35083505641faa00c163f9a645c705ec2d
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# metabolomics-feature-scoring

## Summary

Scores and classifies detected LC/MS peaks as artifacts or true peaks using PeakDetective, a semisupervised deep learning approach combining an unsupervised autoencoder with an actively learned classifier, to curate XCMS peak lists from untargeted metabolomics data.

## When to use

Use when you have an XCMS-generated peak list (mz/rt columns) from LC/MS metabolomics data plus raw mzML files, and need to separate true peaks from artifacts like noise or co-eluting degeneracies before downstream statistics.

## When NOT to use

- When you have only a feature intensity table with no raw mzML data — PeakDetective extracts low-dimensional latent representations from raw extracted ion chromatograms and cannot score peaks without the underlying chromatographic scans
- When the peak list has not been generated yet and you need peak detection itself rather than curation — this skill assumes an existing XCMS peak list as input, though PeakDetective can also run XCMS via detection_helper.PeakList().runXCMS
- When you need fully automated classification with zero user input and cannot afford the 5–10 active learning labeling rounds (or loading a pretrained model) — PeakDetective is semisupervised and requires labeled peaks to train

## Inputs

- Raw LC/MS data in mzML format
- XCMS-generated peak list CSV with mz and rt columns (peaks_formatted.csv)
- Optional pretrained PeakDetective weights in a PeakDetectiveObject directory
- User labels for a small set of peaks during active learning iterations (numValPeaks=10, numPeaksPerRound=10)

## Outputs

- Curated peak list (peak_curated / peaks_curated.csv)
- Per-peak artifact/true-peak classification scores (peak_scores.csv)
- Peak intensities matrix (peak_intensities.csv)
- Trained PeakDetective model saved to a PeakDetectiveObject directory

## How to apply

Load raw mzML files into PeakDetective.rawData objects with a 25.5 ppm ms1 tolerance, then read the XCMS peak list (mz, rt columns from peaks_formatted.csv). Initialize a PeakDetective object (numCores=10, resolution=60), generate extracted ion chromatogram data matrices via makeDataMatrix with RT alignment (align=True), and train the smoother (10 epochs) and classifier with active learning — labeling ~10 peaks per round for 5–10 iterations until a bimodal score histogram emerges and val_mean_absolute_error drops dramatically. Set a classification cutoff (0.8 in the example) based on the FDR and TPR curves reported from validation peaks, then run integ.curatePeaks(raw_data, peaklist, threshold=cutoff, align=True) to produce peak_curated, peak_scores, and peak_intensities outputs. The active-learning loop requires fewer than 100 user-labeled peaks and completes in minutes; the trained model can be saved to a PeakDetectiveObject directory and reloaded for reuse.

## Related tools

- **PeakDetective** (Primary scoring/curation engine: trains an autoencoder latent representation and an active-learning classifier over EIC data matrices, then applies curatePeaks to score each XCMS-detected peak as artifact or true peak) — https://github.com/pattilab/PeakDetective
- **XCMS** (Upstream peak detection tool whose peak list (peaks_formatted.csv) serves as the input that PeakDetective curates; can be invoked via detection_helper.PeakList().runXCMS)
- **pyteomics** (Required dependency used by PeakDetective for reading raw mzML files into rawData objects)
- **numpy** (Required dependency for numerical operations on EIC data matrices and peak score arrays)

## Examples

```
peak_curated,peak_scores,peak_intensities = integ.curatePeaks(raw_data,peaklist,threshold=cutoff,align=True)
```

## Evaluation signals

- After training, the score histogram of all peaks should become bimodal and val_mean_absolute_error should drop dramatically after 5–10 active learning rounds
- FDR and TPR estimates based on the 10 validation peaks should be reported for each cutoff; choose the cutoff (e.g. 0.8) that balances specificity/sensitivity per user preference
- On a spiked-standard P. pastoris dataset, expect a mean 66.7 ± 0.34% of all XCMS peaks and 83.9 ± 0.81% of spiked-metabolite features classified as true peaks (p<0.0001)
- curatePeaks must return three objects (peak_curated, peak_scores, peak_intensities) with the same peak count as the input XCMS peak list
- The saved PeakDetectiveObject directory should allow loading pretrained weights and reproducing classification without retraining

## Limitations

- The classifier is semisupervised and requires active learning: <100 user-labeled peaks across 5–10 iterations; without user labeling the model cannot discriminate artifacts from true peaks
- Autoencoder training requires a large peak pool (min_peaks=100000) for the unsupervised latent representation
- The scoring model depends on data-specific parameters (ms1ppm=25.5, resolution=60, align=True); performance is tailored to specific LC/MS methods and sample types, so a model trained on one dataset may not transfer directly
- Running XCMS or mz.unity degeneracy removal requires R packages installed via installRPackages (>20 min) and R >v4.0 on the path

## Evidence
