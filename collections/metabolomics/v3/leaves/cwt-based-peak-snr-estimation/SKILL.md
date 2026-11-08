---
name: cwt-based-peak-snr-estimation
description: Use when scoring the quality of an already-detected chromatographic peak
  (an intensity vector with apex index and left/right boundaries) to decide whether
  to keep or filter it before downstream peak alignment or annotation in LC/MS metabolomics
  data.
when_to_use_negative:
- Evaluating the quality of an entire extracted ion chromatogram (EIC) with high background
  signals or containing only a single peak — use the EIC-related global zigzag index
  instead, since the paper states the mass chromatographic quality (MCQ) index and
  peak metrics cannot fairly evaluate such EICs.
- Selecting or aligning analyte components between samples — the paper reserves the
  MCQ index for that purpose, not peak-level SNR.
- Peaks whose intensity vectors are shorter than the smallest CWT scale or lack a
  clear apex marker point — the marker-point coefficient is undefined and the ratio
  is unreliable.
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Matlab
  role: Implementation language for the peak quality evaluation and filtering module
    in which the CWT-based SNR is computed (the authors developed their own data processing
    program in Matlab consisting of four sequential modules).
- name: XCMS
  role: Upstream peak detection tool whose chromatographic peak output (R, 'MatchedFilter'
    or 'centWave' options) feeds into the peak quality evaluation where SNR is estimated.
- name: MZmine
  role: Alternative upstream peak detection tool (Java, 'centroidPicker' algorithm)
    whose peak lists are quality-scored with the CWT-based SNR metric.
- name: MAVEN
  role: Alternative upstream peak detection tool (C++, machine learning-based peak
    filtering) whose peak lists are quality-scored with the CWT-based SNR metric.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:2ca16204bf8f3913a8ea292d72dd5e82ccf1dba2d31d68ee87f0adcf1e8b900b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cwt-based-peak-snr-estimation

## Summary

Estimates the signal-to-noise ratio of a detected LC/MS chromatographic peak in the continuous-wavelet-transform domain, as one of six peak quality metrics analyzed by Zhang et al. for filtering 'bad' peaks in metabolomics data.

## When to use

Use when scoring the quality of an already-detected chromatographic peak (an intensity vector with apex index and left/right boundaries) to decide whether to keep or filter it before downstream peak alignment or annotation in LC/MS metabolomics data.

## When NOT to use

- Evaluating the quality of an entire extracted ion chromatogram (EIC) with high background signals or containing only a single peak — use the EIC-related global zigzag index instead, since the paper states the mass chromatographic quality (MCQ) index and peak metrics cannot fairly evaluate such EICs.
- Selecting or aligning analyte components between samples — the paper reserves the MCQ index for that purpose, not peak-level SNR.
- Peaks whose intensity vectors are shorter than the smallest CWT scale or lack a clear apex marker point — the marker-point coefficient is undefined and the ratio is unreliable.

## Inputs

- detected chromatographic peak intensity vector (1D array of intensities from an EIC, with defined apex index and left/right boundaries)
- peak list from a chromatographic peak detection step (e.g. XCMS 'MatchedFilter' or 'centWave', MZmine 'centroidPicker', or MAVEN ML-based filtering)
- choice of CWT mother wavelet and scale-1 configuration for the wavelet-domain noise estimate

## Outputs

- per-peak SNR value (real number; higher = better signal relative to wavelet-domain noise)
- per-peak metric table (e.g. peak_quality_metrics.csv) containing SNR alongside sharpness, Gaussian similarity, peak significance level, TPASR, and local zigzag index
- filtered peak list after applying a user-chosen SNR cutoff threshold

## How to apply

Take the detected peak's intensity vector and apply the continuous wavelet transform (CWT) to it. Compute the 95% quantile of the absolute CWT coefficients at scale 1, which the paper uses as the noise estimate. Locate the marker point at the peak apex and read the CWT coefficient at that point; the SNR is the ratio of this apex marker coefficient to the 95% quantile of the scale-1 coefficients. Zhang et al. compared this CWT-domain SNR against five other peak quality metrics (sharpness, Gaussian similarity, peak significance level, triangle peak area similarity ratio, and the local peak-related local zigzag index) and found it was outperformed by sharpness, peak significance level, and the zigzag index; the recommended practice is to use SNR in combination with other metrics rather than alone. Apply the metric to every peak in the peak list and write per-peak SNR values to a table (e.g. peak_quality_metrics.csv) before applying a cutoff threshold to filter peaks.

## Related tools

- **Matlab** (Implementation language for the peak quality evaluation and filtering module in which the CWT-based SNR is computed (the authors developed their own data processing program in Matlab consisting of four sequential modules).)
- **XCMS** (Upstream peak detection tool whose chromatographic peak output (R, 'MatchedFilter' or 'centWave' options) feeds into the peak quality evaluation where SNR is estimated.)
- **MZmine** (Alternative upstream peak detection tool (Java, 'centroidPicker' algorithm) whose peak lists are quality-scored with the CWT-based SNR metric.)
- **MAVEN** (Alternative upstream peak detection tool (C++, machine learning-based peak filtering) whose peak lists are quality-scored with the CWT-based SNR metric.)

## Examples

```
matlab -batch "peakSnr = cwtApexCoeff(peakIntensity) / quantile(abs(cwt(peakIntensity,'scale',1)), 0.95);"
```

## Evaluation signals

- Smooth Gaussian-like synthetic peaks must receive markedly higher SNR values than zigzag-shaped peaks spanning a range of SNR values, consistent with the paper's finding that good peaks have relatively high SNR.
- The scale-1 noise estimate must be the 95% quantile of the absolute CWT coefficients, not the median or maximum — verify this in the implementation.
- The marker point for the numerator must be at the peak apex, not at an arbitrary scan position.
- SNR values should be finite and positive for every peak in the peak list; NaN or zero denominator indicates the scale-1 coefficient set is degenerate.
- When used in combination filtering, varying the SNR cutoff threshold individually and in combination with other metrics must produce measurable Recall, Precision, and F-Score changes as the paper's evaluation framework requires.

## Limitations

- The paper found SNR does not outperform sharpness, peak significance level, or the zigzag index, because of the zigzag nature of LC/MS chromatographic peaks; using it alone is less efficient than combining several metrics.
- The paper did not establish optimal cutoff thresholds for any quality metric — 'setting an optimal cutoff threshold for each metric still is a challenge task'.
- Ground truth peaks were defined as peaks detected by at least two of XCMS, MZmine, and MAVEN, so SNR evaluation against a single-tool peak list has no validated reference.
- The paper's Matlab scripts and LC/MS profile dataset were only available upon request; no public repository, license, or reference implementation was provided.

## Evidence

- [other] Implement SNR in the wavelet domain as the ratio of the continuous wavelet transform (CWT) coefficient at a marker point to the 95% quantile of the absolute CWT coefficients at scale 1.: [evidence span withheld — anchor status: not_found]
- [intro] We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio: "We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio"
- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
