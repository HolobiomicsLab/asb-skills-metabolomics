---
name: chromatographic-quality-metric-implementation
description: Use when you must programmatically score the quality of extracted ion
  chromatograms (EICs) or detected chromatographic peaks in LC/MS metabolomics data
  — especially EICs with high background signals or containing only a single peak
  — before downstream peak detection.
when_to_use_negative:
- Evaluating EICs selected for analyte component alignment where the MCQ index is
  appropriate — the paper notes the MCQ index is suited for selecting and aligning
  analyte components, but fails on EICs with high background signals or only a single
  peak, which is exactly where the zigzag index applies
- Centroided/peak-picked data with no extracted continuous ion chromatogram intensity
  profile — the first-order derivative computation requires an EIC intensity series
- Automated machine learning-based peak identification — the paper reports SVM-based
  identification is not yet implemented because a curated training set of 'good' and
  'bad' representative peaks does not yet exist
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Matlab
  role: Implementation language for the authors' four-module data processing program
    (acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak
    detection, peak quality evaluation and filtering); the zigzag index can be re-implemented
    in Python or Matlab
- name: XCMS
  role: R-based peak detection tool (MatchedFilter and centWave options) whose peak
    lists serve as one of three ground-truth sources for evaluating metric cutoffs
- name: MZmine
  role: Java-based peak detection tool (centroidPicker algorithm) whose peak lists
    serve as one of three ground-truth sources for evaluating metric cutoffs
- name: MAVEN
  role: C++ peak detection tool with machine learning-based peak filtering whose peak
    lists serve as one of three ground-truth sources for evaluating metric cutoffs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:502562fadee3f6fbcb7727211c3d89bae70d77805b26b01b6e9f0f07ee0f72d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - Python
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chromatographic-quality-metric-implementation

## Summary

Implements and validates chromatographic quality metrics from Zhang & Zhao (2014), most notably the EIC-related global zigzag index computed from an EIC's first-order derivatives, plus six chromatographic peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) used to filter 'bad' EICs and peaks in LC/MS-based metabolomics data.

## When to use

Use when you must programmatically score the quality of extracted ion chromatograms (EICs) or detected chromatographic peaks in LC/MS metabolomics data — especially EICs with high background signals or containing only a single peak — before downstream peak detection.

## When NOT to use

- Evaluating EICs selected for analyte component alignment where the MCQ index is appropriate — the paper notes the MCQ index is suited for selecting and aligning analyte components, but fails on EICs with high background signals or only a single peak, which is exactly where the zigzag index applies
- Centroided/peak-picked data with no extracted continuous ion chromatogram intensity profile — the first-order derivative computation requires an EIC intensity series
- Automated machine learning-based peak identification — the paper reports SVM-based identification is not yet implemented because a curated training set of 'good' and 'bad' representative peaks does not yet exist

## Inputs

- Extracted ion chromatogram (EIC) intensity arrays from LC/MS profile data (e.g. from XCMS, MZmine, or MAVEN processing of an Agilent HPLC / Q-TOF Premier dataset)
- Detected chromatographic peak regions (intensity sub-arrays around peak apexes)
- Metric cutoff thresholds for filtering 'bad' EICs and peaks

## Outputs

- Per-EIC global zigzag index value (dynamic range 0–4.0)
- Per-peak quality metric values: sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index
- Filtered EIC list and filtered peak list retaining 'good'-quality objects for downstream analysis
- Recall, Precision, and F-Score when thresholds are evaluated against ground-truth peaks (peaks detected by at least two of XCMS, MZmine, MAVEN)

## How to apply

Implement the EIC-related global zigzag index from the EIC's first-order increments d_n = I_n − I_{n−1}: compute the summed second-difference term Σ(2I_n − I_{n−1} − I_{n+1})², divide by N × EPI² where EPI is the effective peak intensity (apex intensity minus baseline), giving an index valid over the dynamic range 0–4.0. Apply it as step 2) of the paper's four-module workflow — acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering — so 'bad' EICs are filtered before the time-consuming peak detection step. For detected peaks, compute sharpness, peak significance level, and the local zigzag index, which the authors found outperform the other metrics because of the zigzag nature of LC/MS chromatographic peaks, and prefer combining several metrics over any single one. Validate against synthetic EICs (normal, high-background, single-peak) with a fixed seed, asserting the zigzag index stays within 0–4.0 and that high-background/single-peak cases yield low (good-quality) values. Choose a cutoff threshold empirically, since the paper reports that setting an optimal cutoff for each metric remains a challenge; evaluate candidate thresholds with Recall, Precision, and F-Score.

## Related tools

- **Matlab** (Implementation language for the authors' four-module data processing program (acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering); the zigzag index can be re-implemented in Python or Matlab)
- **XCMS** (R-based peak detection tool (MatchedFilter and centWave options) whose peak lists serve as one of three ground-truth sources for evaluating metric cutoffs)
- **MZmine** (Java-based peak detection tool (centroidPicker algorithm) whose peak lists serve as one of three ground-truth sources for evaluating metric cutoffs)
- **MAVEN** (C++ peak detection tool with machine learning-based peak filtering whose peak lists serve as one of three ground-truth sources for evaluating metric cutoffs)

## Examples

```
import numpy as np; zig = ((2*I[1:-1]-I[:-2]-I[2:])**2).sum()/(len(I)*(I.max()-baseline)**2)  # EIC global zigzag index, range 0-4.0
```

## Evaluation signals

- Global zigzag index values fall within the paper's dynamic range of 0–4.0 for all evaluated EICs
- EICs with high background signals and EICs containing only a single peak yield low (good-quality) zigzag index values, matching the paper's reported robustness
- Reproducibility: with a fixed random seed and documented peak width/height/noise parameters, re-running the synthetic EIC generator reproduces identical index values (deterministic script)
- When cutoff thresholds are varied, Recall/Precision/F-Score against ground-truth peaks (detected by ≥2 of XCMS, MZmine, MAVEN) quantify filtering quality
- Sharpness, peak significance level, and zigzag index consistently outperform the remaining three peak metrics on chromatographic peaks, consistent with the zigzag nature of LC/MS peaks

## Limitations

- Setting an optimal cutoff threshold for each quality metric remains an unsolved challenge; thresholds must be tuned empirically
- The original LC/MS profile dataset and Matlab scripts are available only upon request; no public repository or license is provided ('License: None for usage')
- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak — use the global zigzag index for those scenarios instead
- Machine learning-based automatic peak identification (e.g., SVM) is not yet implemented because a curated training set of good and bad representative peaks does not yet exist

## Evidence

- [methods] EIC quality evaluation and filtering precedes peak detection in the four-module workflow: "which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [abstract] The EIC-related global zigzag index is robust for high-background and single-peak EICs: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] Sharpness, peak significance level, and zigzag index are the best-performing peak metrics: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [discussion] Optimal cutoff thresholds remain unsolved: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [abstract] Combining metrics beats individual metrics: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] Ground truth defined by multi-tool agreement: "a peak is considered a ground truth peak if it can be detected by multiple tools"
