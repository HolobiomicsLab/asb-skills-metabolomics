---
name: triangle-peak-area-similarity-ratio-computation
description: Use when scoring detected chromatographic peaks in LC/MS-based metabolomics
  data where a peak's intensity vector, apex index, and left/right boundaries are
  available, and symmetry/shape quality must be quantified for downstream peak filtering.
when_to_use_negative:
- Input is an EIC (not an individual detected peak) — for EIC quality use the MCQ
  index or the EIC-related global zigzag index, since the local zigzag/TPASR metrics
  target detected peaks
- You expect a standalone quality verdict — the paper reports sharpness, peak significance
  level, and zigzag index outperform Gaussian similarity and TPASR due to the zigzag
  nature of LC/MS peaks, so TPASR alone is a weaker discriminator
- No apex index or left/right peak boundaries are defined for the peak, making TPA
  and RPA undefined
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Language in which the authors' four-module data processing program, including
    the peak quality evaluation and filtering module that computes TPASR, was developed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:60a60479870494425ae4aba544ddc6e7f3e157a0aeb82298052b2e6a03ceda11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# triangle-peak-area-similarity-ratio-computation

## Summary

Computes the triangle peak area similarity ratio (TPASR), one of six chromatographic peak quality metrics from Zhang & Zhao (2014), which scores how well a detected LC/MS peak's raw area matches the idealized triangle formed by its peak width and apex intensity.

## When to use

Use when scoring detected chromatographic peaks in LC/MS-based metabolomics data where a peak's intensity vector, apex index, and left/right boundaries are available, and symmetry/shape quality must be quantified for downstream peak filtering.

## When NOT to use

- Input is an EIC (not an individual detected peak) — for EIC quality use the MCQ index or the EIC-related global zigzag index, since the local zigzag/TPASR metrics target detected peaks
- You expect a standalone quality verdict — the paper reports sharpness, peak significance level, and zigzag index outperform Gaussian similarity and TPASR due to the zigzag nature of LC/MS peaks, so TPASR alone is a weaker discriminator
- No apex index or left/right peak boundaries are defined for the peak, making TPA and RPA undefined

## Inputs

- Detected chromatographic peak profiles as per-peak intensity vectors with a defined apex index and left/right peak boundaries (e.g., from XCMS, MZmine, or MAVEN peak lists on a UPLC/MS dataset)
- Per-peak metadata: peak width and apex intensity

## Outputs

- Per-peak TPASR value per detected chromatographic peak (one row per peak, lower values indicate higher quality)
- Per-peak metric value table combining TPASR with sharpness, Gaussian similarity, SNR, peak significance level, and local zigzag index (e.g., peak_quality_metrics.csv)

## How to apply

For each detected peak, take its raw intensity vector between the defined left and right boundaries. Compute the triangle peak area TPA = 0.5 × peak width × apex intensity, and the raw peak area RPA as the summed intensity between the boundaries; TPASR is then |TPA − RPA| / TPA (equation 8 in the paper). TPASR is one of six metrics implemented in the authors' Matlab peak quality evaluation and filtering module, alongside sharpness, Gaussian similarity, SNR (CWT-based, 95% quantile of scale-1 coefficients), peak significance level, and the local zigzag index. Lower TPASR denotes higher-quality, more symmetric peaks, so apply it with a cutoff threshold before downstream processing; combine it with the other five metrics since the paper found metric combinations outperform individual metrics. Validation uses a deterministic synthetic peak set: smooth Gaussian-like peaks must score low TPASR while zigzag-shaped peaks score markedly higher.

## Related tools

- **Matlab** (Language in which the authors' four-module data processing program, including the peak quality evaluation and filtering module that computes TPASR, was developed)

## Examples

```
matlab -batch "TPA=0.5*width*apexInt; RPA=sum(intens(lb:rb)); TPASR=abs(TPA-RPA)/TPA; writemeterix" ; % TPASR per peak per equation 8, output to peak_quality_metrics.csv
```

## Evaluation signals

- TPASR values are bounded in [0, 1] since |TPA − RPA| ≤ TPA when RPA ≤ 2×TPA; out-of-range values indicate boundary or apex-intensity errors
- On a deterministic synthetic peak set (fixed seed), smooth Gaussian-like peaks must score low TPASR while zigzag-shaped peaks must receive markedly higher TPASR values
- RPA must equal the summed raw intensity strictly between the defined peak boundaries, and TPA must equal 0.5 × peak width × apex intensity
- The per-peak metric table (e.g., peak_quality_metrics.csv) must contain one TPASR value per detected peak, aligned with the other five metric columns

## Limitations

- Setting an optimal cutoff threshold for TPASR remains an unsolved challenge; thresholds must be varied individually and in combination with other metrics
- Of the six metrics, sharpness, peak significance level, and zigzag index outperform the others (including TPASR) for LC/MS peaks, so TPASR alone is insufficient for peak filtering
- The paper's Matlab scripts and LC/MS profile dataset are available only upon request; no public repository is provided

## Evidence

- [abstract] Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks.: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [intro] We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio: "We also analyzed and compared a comprehensive set of chromatographic peak quality metrics including sharpness, Gaussian similarity, SNR, peak significance level, triangle peak area similarity ratio"
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
