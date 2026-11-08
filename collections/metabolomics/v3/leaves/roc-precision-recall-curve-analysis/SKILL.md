---
name: roc-precision-recall-curve-analysis
description: Use when comparing NMR preprocessing/summarization methods (peak-picking-then-grouping
  vs binning vs alignment+binning) on case-vs-control spectra and you need ROC and
  Precision-Recall curves as the performance metric.
when_to_use_negative:
- Input is already a feature table or the analysis goal is a single method's performance
  rather than comparing spectral preprocessing/summarization alternatives
- The spectra are not labeled into case vs control groups, since ROC/PRC curves require
  known class membership
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: speaq 2.0
  role: Peak-based (wavelet peak finding + grouping) summarization method being benchmarked
    against alignment-plus-binning
  repo: https://github.com/beirnaert/speaq
- name: speaq R package
  role: R implementation providing getWaveletPeaks, PeakGrouper and BuildFeatureMatrix
    used to build the speaq 2.0 feature matrix
  repo: https://cran.r-project.org/package=speaq
- name: CluPA
  role: Spectral alignment method used as a competing approach, followed by binning
- name: icoshift
  role: Spectral alignment method used as a competing approach, followed by binning
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1006018
    title: 'speaq 2.0: A complete workflow for high-throughput 1D NMR spectra processing
      and quantification'
  source_content_hash: sha256:979fff9b909d1d07dab50b4401ccc13c5c4160789b43163be322c3d6cd59d505
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beirnaert/speaq
  tools:
  - CluPA
  - R
  - icoshift
  - speaq 2.0
  - speaq R package
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# roc-precision-recall-curve-analysis

## Summary

Benchmarking method that evaluates 1D NMR spectral summarization workflows (speaq 2.0 peak-based processing vs binning and alignment-plus-binning) by computing ROC and Precision-Recall curves from case-vs-control classification of the resulting feature matrices, as used in the speaq 2.0 simulated-data comparison.

## When to use

Use when comparing NMR preprocessing/summarization methods (peak-picking-then-grouping vs binning vs alignment+binning) on case-vs-control spectra and you need ROC and Precision-Recall curves as the performance metric.

## When NOT to use

- Input is already a feature table or the analysis goal is a single method's performance rather than comparing spectral preprocessing/summarization alternatives
- The spectra are not labeled into case vs control groups, since ROC/PRC curves require known class membership

## Inputs

- Matrix of 20 simulated 1H NMR spectra (10 case, 10 control) built from HMDB0000440 and HMDB0001336 reference spectra with noise, baseline distortion, and random peak shifts
- PPM axis vector for the spectra
- Case-vs-control class factor/labels
- Feature matrices produced by speaq 2.0 peak grouping and by binning (raw and post-CluPA/icoshift alignment)

## Outputs

- ROC curves per summarization method
- Precision-Recall curves per summarization method
- Combined comparison plot (ROC_PRC_curves.png)

## How to apply

Build one feature matrix per competing summarization method: for speaq 2.0 run Mexican hat wavelet peak finding followed by peak grouping (getWaveletPeaks, PeakGrouper, BuildFeatureMatrix), avoiding binning; for the baselines use binning of raw shifted spectra, and binning after CluPA or icoshift alignment. On the simulated case-vs-control dataset (20 spectra — 10 case, 10 control — built from two HMDB 1H NMR metabolites, HMDB0000440 and HMDB0001336, with added noise, baseline distortion, and random peak shifts per S2 Appendix), classify case vs control from each feature matrix and compute ROC curves and Precision-Recall curves. Plot all methods' curves together (e.g. ROC_PRC_curves.png) and judge the peak-based method correct if it performs similar to or better than the alignment-plus-binning alternatives.

## Related tools

- **speaq 2.0** (Peak-based (wavelet peak finding + grouping) summarization method being benchmarked against alignment-plus-binning) — https://github.com/beirnaert/speaq
- **speaq R package** (R implementation providing getWaveletPeaks, PeakGrouper and BuildFeatureMatrix used to build the speaq 2.0 feature matrix) — https://cran.r-project.org/package=speaq
- **CluPA** (Spectral alignment method used as a competing approach, followed by binning)
- **icoshift** (Spectral alignment method used as a competing approach, followed by binning)

## Examples

```
peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector); groups <- PeakGrouper(Y.peaks = peaks); Features <- BuildFeatureMatrix(Y.data = groups)
```

## Evaluation signals

- The speaq 2.0 peak-based ROC/PRC curve should lie on or above the curves for binning of raw shifted spectra and binning after CluPA or icoshift alignment, consistent with 'performs similar or better compared to the other methods'
- The simulated dataset must contain exactly 20 spectra split 10 case / 10 control, derived from HMDB0000440 and HMDB0001336
- Each method must yield a feature/sample/peak-value matrix suitable for case-vs-control classification without a binning step for the speaq 2.0 arm
- All methods' ROC and Precision-Recall curves must be plotted together in a single comparison figure (ROC_PRC_curves.png)

## Limitations

- Spectral alignment algorithms are not always optimal when dealing with severely shifted spectra, which affects subsequent steps such as binning
- In severe-shift cases the speaq 2.0 algorithm may delete one of the large peaks in the grouping step; the optional SilhouetR function can detect this problem via silhouette values per group
- Binning suffers from difficult bin boundary definition in crowded spectra and intrinsic information loss since entire bins are summed together

## Evidence

- [supplementary] Simulated data based comparison of speaq 2.0 vs alignment algorithms.: "Simulated data based comparison of speaq 2.0 vs alignment algorithms."
- [supplementary] speaq 2.0 peak-based method performs similar or better than alternatives: "The speaq 2.0 method, based on finding peaks and subsequently grouping, performs similar or better compared to the other methods"
- [abstract] peak-picking followed by grouping, avoiding binning: "starts with the common approach of peak-picking, followed by grouping, thus avoiding the binning step"
- [supplementary] alignment results not always optimal for severely shifted spectra: "The results of spectral alignment algorithms is not always optimal when dealing with severely shifted spectra."
- [readme] README minimal example showing peak detection, grouping and feature matrix construction: "peaks <- getWaveletPeaks(Y.spec = spectra.matrix, X.ppm = ppm.vector)  
# peak grouping
groups <- PeakGrouper(Y.peaks = peaks)"
