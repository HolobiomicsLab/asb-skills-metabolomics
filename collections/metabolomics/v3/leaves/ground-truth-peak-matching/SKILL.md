---
name: ground-truth-peak-matching
description: Use when evaluating chromatographic peak quality metrics on LC/MS data
  and no curated reference peak set exists, so a consensus-based ground truth must
  be derived from multiple independent peak-detection tools before computing TP/Recall/Precision/F-Score.
when_to_use_negative:
- A curated ground truth or manually validated peak list already exists — consensus
  of three detectors is only a proxy when no curated reference is available
- The task is EIC quality evaluation (e.g., MCQ index or global zigzag index scoring
  of extracted ion chromatograms) rather than detected chromatographic peak quality
  evaluation
- Evaluating tools whose peak lists feed the consensus itself (XCMS/MZmine/MAVEN)
  against this ground truth — circularity invalidates the comparison
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: XCMS
  role: One of the three peak-detection tools whose individual peak lists vote to
    form the ground truth (R package, MatchedFilter and centWave options)
- name: MZmine
  role: One of the three peak-detection tools contributing peak lists to the ground
    truth consensus (Java, centroidPicker algorithm)
- name: MAVEN
  role: One of the three peak-detection tools contributing peak lists to the ground
    truth consensus (C++, machine learning-based peak filtering)
- name: Matlab
  role: Language of the four-module processing program (acute EIC extraction, EIC
    quality evaluation/filtering, peak detection, peak quality evaluation/filtering)
    whose retained peaks are matched against the ground truth list
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:7faf03fd63675f750b16a35186f0b307326aa5b55277b412093e289c09c162ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ground-truth-peak-matching

## Summary

Establishes a ground truth chromatographic peak list by taking the consensus of peaks detected by at least two of XCMS, MZmine, and MAVEN on the same LC/MS metabolomics dataset, then matches candidate peaks against it (0.05 Da m/z tolerance, 5 s retention-time tolerance) to compute Recall, Precision, and F-Score for peak quality metrics.

## When to use

Use when evaluating chromatographic peak quality metrics on LC/MS data and no curated reference peak set exists, so a consensus-based ground truth must be derived from multiple independent peak-detection tools before computing TP/Recall/Precision/F-Score.

## When NOT to use

- A curated ground truth or manually validated peak list already exists — consensus of three detectors is only a proxy when no curated reference is available
- The task is EIC quality evaluation (e.g., MCQ index or global zigzag index scoring of extracted ion chromatograms) rather than detected chromatographic peak quality evaluation
- Evaluating tools whose peak lists feed the consensus itself (XCMS/MZmine/MAVEN) against this ground truth — circularity invalidates the comparison

## Inputs

- LC/MS profile dataset from an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer
- Individual detected-peak lists produced by XCMS, MZmine, and MAVEN on the same dataset
- Detected chromatographic peak list with six quality metric values (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index)

## Outputs

- Ground truth peak list (peaks detected by at least two of XCMS, MZmine, and MAVEN)
- True positive counts per metric/threshold setting via TP(X)=N(X∩TruePeaklist)
- Recall, Precision, and F-Score values per metric and for the combined-metric configuration (expected combined values: Recall = 0.7076, Precision = 0.6186, F-Score = 0.6601)
- F-Score versus threshold curves for the six peak quality metrics

## How to apply

Run three open-source peak-detection tools — XCMS (R, MatchedFilter/centWave), MZmine (Java, centroidPicker), and MAVEN (C++, machine-learning-based peak filtering) — on the same LC/MS profile dataset and collect each tool's individual peak list. Declare a peak a ground truth peak if it is detected by at least two of the three tools, matching peaks across tools with a 0.05 Da m/z tolerance and a 5 second retention-time tolerance. Then, for each peak quality metric being evaluated (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index), vary its cutoff threshold, retain peaks passing the cutoff, and compute true positives per TP(X)=N(X∩TruePeaklist) by matching the retained set against the ground truth list. At each threshold compute Recall, Precision, and F-Score, both per metric and for the six metrics combined at the joint thresholds (Sharpness_Th = 2.0, Gaussian_Similarity_Th = 0.6, SNR_Th = 1.3, Peak_Significance_Th = 1.2, TPASR_Th = 0.8, zigzag index = 0.9); the rationale is that the consensus definition avoids biasing evaluation toward any single detector.

## Related tools

- **XCMS** (One of the three peak-detection tools whose individual peak lists vote to form the ground truth (R package, MatchedFilter and centWave options))
- **MZmine** (One of the three peak-detection tools contributing peak lists to the ground truth consensus (Java, centroidPicker algorithm))
- **MAVEN** (One of the three peak-detection tools contributing peak lists to the ground truth consensus (C++, machine learning-based peak filtering))
- **Matlab** (Language of the four-module processing program (acute EIC extraction, EIC quality evaluation/filtering, peak detection, peak quality evaluation/filtering) whose retained peaks are matched against the ground truth list)

## Evaluation signals

- Ground truth list contains only peaks detected by at least two of XCMS, MZmine, and MAVEN under 0.05 Da m/z / 5 s RT matching
- Combined-metric run reproduces Recall = 0.7076, Precision = 0.6186, F-Score = 0.6601
- Maximum single-metric F-Score (sharpness, peak significance level, or zigzag index best) remains smaller than 0.6, confirming the combined metrics exceed every individual metric
- F-Score curves for sharpness, peak significance level, and zigzag index show clear maxima across the threshold sweep
- Matching uses TP(X)=N(X∩TruePeaklist) consistently across all threshold settings

## Limitations

- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository is provided
- Setting an optimal cutoff threshold for each quality metric remains an unsolved challenge
- A machine-learning-based peak quality evaluator (e.g., SVM) is not yet implemented because no curated training set of good and bad representative peaks exists
- The consensus definition is an approximation of true peaks — a real missed peak absent from all three tools cannot be recovered

## Evidence

- [methods] Ground truth peak definition via multi-tool consensus: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [methods] Three tools used to build consensus on the same dataset: "We used the three tools to analyze the same UPLC/MS dataset and produced individual peak lists."
- [abstract] Combined metrics outperform individual metrics in peak quality evaluation: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [intro] Threshold sweep with Recall, Precision, F-Score as the evaluation protocol: [evidence span withheld — anchor status: not_found]
