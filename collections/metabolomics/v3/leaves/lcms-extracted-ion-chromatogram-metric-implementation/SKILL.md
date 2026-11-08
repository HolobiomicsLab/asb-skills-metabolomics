---
name: lcms-extracted-ion-chromatogram-metric-implementation
description: Use when you have LC/MS or UPLC/MS profile intensity profiles (extracted
  ion chromatograms) and need to score each EIC or detected peak's quality with quantitative
  metrics, or to filter out high-background / single-peak / spike / zigzag noise before
  time-consuming peak detection processing.
when_to_use_negative:
- Input is already a list of detected peaks or a feature table rather than raw EIC
  intensity profiles — use downstream peak-quality metrics only, not EIC-level MCQ/zigzag
  computation.
- Data are not chromatographic (e.g. MS-only, no retention-time intensity profile),
  so first-order derivative zigzag metrics over neighboring data points are not meaningful.
- You need fully automatic peak identification via machine learning (e.g. SVM); the
  article notes such an approach requires a curated training set of good/bad representative
  peaks that does not yet exist.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Matlab
  role: Programming language used to implement the custom four-module data processing
    program (acute EIC extraction, EIC quality evaluation and filtering, chromatographic
    peak detection, peak quality evaluation and filtering), including MCQ and zigzag
    index computation.
- name: XCMS
  role: R-based peak detection tool used to generate one of the individual peak lists;
    consensus of its peaks with MZmine and MAVEN defines ground truth peaks for metric
    evaluation.
- name: MZmine
  role: Java-based peak detection tool (centroidPicker algorithm) used to generate
    one of the individual peak lists for ground truth definition.
- name: MAVEN
  role: C++-based peak detection tool with machine learning-based peak filtering,
    used to generate one of the individual peak lists for ground truth definition.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:efbd6f23d08d166bbe7a1de2f6b5678470919286784296469c9d35f5efc927d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lcms-extracted-ion-chromatogram-metric-implementation

## Summary

Implement and compute EIC quality metrics (MCQ index and EIC-related global zigzag index from first-order derivatives) plus chromatographic peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index) for LC/MS metabolomics profile data, to filter out 'bad' EICs and peaks before downstream peak detection.

## When to use

Use when you have LC/MS or UPLC/MS profile intensity profiles (extracted ion chromatograms) and need to score each EIC or detected peak's quality with quantitative metrics, or to filter out high-background / single-peak / spike / zigzag noise before time-consuming peak detection processing.

## When NOT to use

- Input is already a list of detected peaks or a feature table rather than raw EIC intensity profiles — use downstream peak-quality metrics only, not EIC-level MCQ/zigzag computation.
- Data are not chromatographic (e.g. MS-only, no retention-time intensity profile), so first-order derivative zigzag metrics over neighboring data points are not meaningful.
- You need fully automatic peak identification via machine learning (e.g. SVM); the article notes such an approach requires a curated training set of good/bad representative peaks that does not yet exist.

## Inputs

- LC/MS profile dataset (e.g. from an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer), as raw intensity profiles
- Extracted ion chromatograms (EICs) as intensity profiles over retention time
- Detected chromatographic peak lists from XCMS, MZmine, and MAVEN (for ground truth comparison)
- Matlab scripts implementing the four-module pipeline

## Outputs

- Per-EIC metric values: MCQ index and EIC-related global zigzag index (range [0, 4.0])
- Per-peak metric values: sharpness, Gaussian similarity, SNR, peak significance level, TPASR, and local peak-related zigzag index
- Filtered 'good'/'bad' EIC and peak lists after applying cutoff thresholds
- Recall, Precision, and F-Score tables when metric cutoff thresholds are varied individually and in combination

## How to apply

Build on the four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering. For EIC quality, implement the mass chromatographic quality (MCQ) index by measuring similarity of each original mass chromatogram to its smoothed version (spike detection) and to its mean-subtracted version (background detection); then implement the EIC-related global zigzag index from first-order derivatives as the sum of (2*In - In-1 - In+1)^2 over neighboring data points, normalized by N times the squared effective peak intensity (max intensity minus baseline at apex), yielding a value in [0, 4.0]. For peak quality, compute sharpness, Gaussian similarity, SNR, peak significance level, TPASR (triangle peak area similarity ratio), and local peak-related zigzag index; the article reports that sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks, and that using several metrics in combination is more efficient than individual metrics. Note the known failure mode: the MCQ index cannot fairly evaluate EICs with high background signals or containing only a single peak, while the global zigzag index is robust in both scenarios — so use the global zigzag index when high-background or single-peak EICs are present. Ground truth peaks for evaluation can be defined as peaks detected by at least two of XCMS, MZmine, and MAVEN.

## Related tools

- **Matlab** (Programming language used to implement the custom four-module data processing program (acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering), including MCQ and zigzag index computation.)
- **XCMS** (R-based peak detection tool used to generate one of the individual peak lists; consensus of its peaks with MZmine and MAVEN defines ground truth peaks for metric evaluation.)
- **MZmine** (Java-based peak detection tool (centroidPicker algorithm) used to generate one of the individual peak lists for ground truth definition.)
- **MAVEN** (C++-based peak detection tool with machine learning-based peak filtering, used to generate one of the individual peak lists for ground truth definition.)

## Evaluation signals

- The EIC-related global zigzag index must be normalized to the range [0, 4.0].
- For known high-background or single-peak EICs (e.g. EIC_IDs 27, 90, 134, 154), the global zigzag index should remain low (locally good) even though the MCQ index is low.
- For a 'bad' spiky EIC (e.g. EIC_ID 167), the MCQ index should be low and the global zigzag index high.
- When varying cutoff thresholds individually and in combination, Recall, Precision, and F-Score should be reported for each metric and metric combination.
- Ground truth peaks must be reproducible as the intersection of peaks detected by at least two of XCMS, MZmine, and MAVEN.

## Limitations

- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository is provided.
- No license is specified for the software ('None for usage').
- Setting an optimal cutoff threshold for each quality metric still is a challenging task.
- Machine learning-based automatic peak identification (e.g. SVM) is not implemented; it requires curating a sufficient number of 'good' and 'bad' representative peaks that does not yet exist.
- The MCQ index cannot fairly evaluate EICs with high background signals or containing only a single peak.

## Evidence

- [methods] Four-module pipeline in Matlab: "we developed our own data processing program in Matlab, which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection,"
- [abstract] Global zigzag index normalization: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] Best-performing peak metrics: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] MCQ index limitation: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [methods] Ground truth definition: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] Availability limitation: "Availability: The LC/MS profile dataset and Matlab scripts are available upon request."
