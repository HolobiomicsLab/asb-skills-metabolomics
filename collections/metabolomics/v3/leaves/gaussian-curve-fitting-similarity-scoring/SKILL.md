---
name: gaussian-curve-fitting-similarity-scoring
description: Use when detected LC/MS chromatographic peaks must be quality-filtered
  by shape symmetry, e.g. peaks that look Gaussian-like versus zigzag, distorted,
  or asymmetric profiles after peak detection.
when_to_use_negative:
- Input is a raw LC/MS profile (mzML/NetCDF) with no peaks detected yet — run acute
  EIC extraction and peak detection first; this skill scores already-detected peaks.
- Evaluating the quality of the full extracted ion chromatogram rather than a single
  detected peak — use the EIC-related global zigzag index (or MCQ index) instead of
  the peak-level Gaussian similarity metric.
- Peaks with high background signals or containing only a single point where curve
  fitting is unstable — the article notes the MCQ index cannot fairly evaluate such
  EICs, and Gaussian fitting is equally unreliable on degenerate profiles.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Matlab
  role: Implementation language for the data processing program that computes the
    Gaussian similarity metric on detected chromatographic peaks
- name: XCMS
  role: Peak detection source (R, 'MatchedFilter' and 'centWave') producing the peak
    lists that Gaussian similarity is scored against
- name: MZmine
  role: Peak detection source (Java, 'centroidPicker') producing ground truth peak
    lists used to calibrate quality-metric thresholds
- name: MAVEN
  role: Peak detection source (C++, machine learning-based peak filtering) contributing
    ground truth peaks for metric evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:e9bc8efb817e0b80802b7d95303fe3a6fded7d68fc02655f620fe26f5fa234c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gaussian-curve-fitting-similarity-scoring

## Summary

Scores the symmetry and shape quality of an LC/MS chromatographic peak by fitting a classical or modified Gaussian curve to the detected peak intensities and computing the dot-product similarity between the observed profile and the fitted curve.

## When to use

Use when detected LC/MS chromatographic peaks must be quality-filtered by shape symmetry, e.g. peaks that look Gaussian-like versus zigzag, distorted, or asymmetric profiles after peak detection.

## When NOT to use

- Input is a raw LC/MS profile (mzML/NetCDF) with no peaks detected yet — run acute EIC extraction and peak detection first; this skill scores already-detected peaks.
- Evaluating the quality of the full extracted ion chromatogram rather than a single detected peak — use the EIC-related global zigzag index (or MCQ index) instead of the peak-level Gaussian similarity metric.
- Peaks with high background signals or containing only a single point where curve fitting is unstable — the article notes the MCQ index cannot fairly evaluate such EICs, and Gaussian fitting is equally unreliable on degenerate profiles.

## Inputs

- Detected chromatographic peak intensity vector with apex index and left/right boundary indices (from XCMS, MZmine, MAVEN, or a custom Matlab peak-detection module)
- Extracted ion chromatogram (EIC) intensity profile from which the peak was detected

## Outputs

- Per-peak Gaussian similarity score (dot product between peak intensities and fitted Gaussian curve)
- Per-peak filter decision after thresholding, combined with sharpness, SNR, peak significance level, TPASR, and local zigzag index

## How to apply

Take each detected peak as a vector of raw intensities spanning its left and right boundaries with a defined apex index, as produced by the acute EIC extraction and chromatographic peak detection modules of an LC/MS metabolomics pipeline (e.g. XCMS 'centWave'/'MatchedFilter', MZmine, MAVEN). Fit either a classical or a modified Gaussian curve to the intensity vector centered at the apex. Compute the Gaussian similarity score as the dot product between the detected peak intensities and the fitted Gaussian curve values: peaks that closely follow a Gaussian shape yield high similarity, while zigzag, shouldered, or asymmetric peaks yield low similarity. Combine the score with the other five quality metrics (sharpness, wavelet-domain SNR, peak significance level, TPASR, local zigzag index) before applying cutoff thresholds, since the paper shows combining several metrics is more effective than any single metric. Note that among the six metrics, sharpness, peak significance level, and zigzag index outperformed Gaussian similarity because of the zigzag nature of LC/MS chromatographic peaks, so treat low Gaussian similarity as one corroborating signal rather than a standalone filter.

## Related tools

- **Matlab** (Implementation language for the data processing program that computes the Gaussian similarity metric on detected chromatographic peaks)
- **XCMS** (Peak detection source (R, 'MatchedFilter' and 'centWave') producing the peak lists that Gaussian similarity is scored against)
- **MZmine** (Peak detection source (Java, 'centroidPicker') producing ground truth peak lists used to calibrate quality-metric thresholds)
- **MAVEN** (Peak detection source (C++, machine learning-based peak filtering) contributing ground truth peaks for metric evaluation)

## Examples

```
matlab -r "load peaks.mat; score = detected_peak_intensities * fitted_gaussian_curve'"
```

## Evaluation signals

- Smooth Gaussian-like synthetic peaks should receive high Gaussian similarity scores, whereas zigzag-shaped peaks should receive markedly lower scores.
- Gaussian similarity ranks below sharpness, peak significance level, and zigzag index in the paper's comparative performance evaluation — do not expect it to be the top-scoring metric.
- When combined with the other five metrics at varied cutoff thresholds, Recall, Precision, and F-Score on ground truth peaks (detected by at least two of XCMS, MZmine, MAVEN) should improve over using Gaussian similarity alone.
- Every detected peak must receive a finite similarity value; failures typically indicate boundary or apex-index errors in the upstream peak vector.

## Limitations

- Setting an optimal cutoff threshold for each quality metric, including Gaussian similarity, remains an unsolved challenge.
- Machine learning-based automatic peak filtering (e.g. SVM) using such metrics requires a curated training set of good and bad representative peaks that does not yet exist.
- The metric underperforms sharpness, peak significance level, and the zigzag index because LC/MS chromatographic peaks are inherently zigzag-shaped.
- The Matlab scripts and LC/MS profile dataset used to validate the metrics were available only upon request; no public reference implementation is cited.

## Evidence

- [intro] We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio: "We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio"
- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation.: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [intro] During data processing, the metric cutoff thresholds were varied, both individually and in combination, and the Recall, Precision, and F-Score of each metric: [evidence span withheld — anchor status: not_found]
- [methods] a peak is considered a ground truth peak if it can be detected by multiple tools: "a peak is considered a ground truth peak if it can be detected by multiple tools"
