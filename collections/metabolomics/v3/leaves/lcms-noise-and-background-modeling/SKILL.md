---
name: lcms-noise-and-background-modeling
description: Use when EICs extracted from LC/MS profile data (e.g. Agilent HPLC Q-TOF
  metabolomics runs) show high background signals or contain only a single peak, and
  a quality score is needed to filter bad EICs prior to time-consuming peak detection.
when_to_use_negative:
- When the input is already a detected chromatographic peak list rather than raw EIC
  intensities — use peak quality evaluation and filtering (sharpness, Gaussian similarity,
  SNR, peak significance level, TPASR, local peak-related zigzag index) instead.
- When only a single EIC quality metric is needed for component selection and alignment
  without concern for high-background or single-peak EICs — the MCQ index alone is
  suited for selecting and aligning analyte components but cannot fairly evaluate
  EICs with high background signals or those containing only a single peak.
- When the goal is automatic machine-learning-based peak identification (e.g. SVM)
  — this requires a curated training set of good and bad representative peaks that
  does not yet exist.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Open-source tool employed to detect meaningful chromatographic peaks and produce
    EICs for quality evaluation
- name: MZmine
  role: Open-source tool employed with the 'centroidPicker' algorithm for chromatographic
    peak detection, producing peak lists used as comparators
- name: MAVEN
  role: Open-source tool equipped with a complex, machine learning-based peak filtering
    method, used as comparator
- name: Matlab
  role: Programming language for the authors' four-module data processing program
    that implements EIC quality evaluation and filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:f59df0378d4f7c8e9aedf0e920010ed9b1346b0c18516feb7b5ff29633dc5ecc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAVEN
  - MZmine
  - Matlab
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lcms-noise-and-background-modeling

## Summary

Quantifies background-signal and single-peak artifacts in extracted ion chromatograms (EICs) from LC/MS metabolomics profiles by modeling EIC noise via the mass chromatographic quality (MCQ) index and its global zigzag index alternative, enabling filtering of bad EICs before downstream chromatographic peak detection.

## When to use

Use when EICs extracted from LC/MS profile data (e.g. Agilent HPLC Q-TOF metabolomics runs) show high background signals or contain only a single peak, and a quality score is needed to filter bad EICs prior to time-consuming peak detection.

## When NOT to use

- When the input is already a detected chromatographic peak list rather than raw EIC intensities — use peak quality evaluation and filtering (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) instead.
- When only a single EIC quality metric is needed for component selection and alignment without concern for high-background or single-peak EICs — the MCQ index alone is suited for selecting and aligning analyte components but cannot fairly evaluate EICs with high background signals or those containing only a single peak.
- When the goal is automatic machine-learning-based peak identification (e.g. SVM) — this requires a curated training set of good and bad representative peaks that does not yet exist.

## Inputs

- Extracted ion chromatogram (EIC) intensities from LC/MS profile data (e.g. from XCMS 'MatchedFilter'/'centWave', MZmine 'centroidPicker', or MAVEN machine learning-based peak filtering outputs)
- Smoothed version of each original EIC (spike detection component)
- Mean-subtracted version of each original EIC (background detection component)

## Outputs

- MCQ index value per EIC (bounded 0–1.0)
- EIC-related global zigzag index value per EIC (based on first order derivatives)
- Filtered EIC list with bad EICs removed before chromatographic peak detection
- Recall, Precision, and F-Score tables when cutoff thresholds are varied individually and in combination

## How to apply

Compute a similarity index between each original EIC and a smoothed version to detect spikes, and a similarity index between the original EIC and its mean-subtracted version to detect background level; combine these two indexes into the MCQ index, bounded in the dynamic range 0–1.0. Run the MCQ function over every EIC in the extracted set (e.g. from XCMS, MZmine, or MAVEN outputs on the same UPLC/MS dataset) and record one MCQ value per EIC. Because the MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak, complement it with the EIC-related global zigzag index, which is based on the EIC's first order derivatives and is robust enough to evaluate EIC qualities in both scenarios. Filter EICs whose quality scores fall below a chosen cutoff threshold before the downstream chromatographic peak detection module.

## Related tools

- **XCMS** (Open-source tool employed to detect meaningful chromatographic peaks and produce EICs for quality evaluation)
- **MZmine** (Open-source tool employed with the 'centroidPicker' algorithm for chromatographic peak detection, producing peak lists used as comparators)
- **MAVEN** (Open-source tool equipped with a complex, machine learning-based peak filtering method, used as comparator)
- **Matlab** (Programming language for the authors' four-module data processing program that implements EIC quality evaluation and filtering)

## Examples

```
mcq_index_values.csv = run_mcq_index(eics, spike_similarity_fn, background_similarity_fn); # bounded 0-1.0 per EIC
```

## Evaluation signals

- All MCQ index values must lie in the dynamic range 0 to 1.0
- Normal EICs must score high on the MCQ index, while EICs with high background signals or containing only a single peak must produce relatively low MCQ indices
- The EIC-related global zigzag index must remain robust (not degrade) on EICs with high background signals or with only a single peak
- When cutoff thresholds are varied individually and in combination, resulting Recall, Precision, and F-Score values should show that sharpness, peak significance level, and zigzag index outperform the other peak quality metrics
- Filtered EIC set should not eliminate peaks that are detected by at least two of XCMS, MZmine, and MAVEN (ground truth peaks)

## Limitations

- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak
- Setting an optimal cutoff threshold for each quality metric still is a challenge task
- Machine learning-based automatic peak identification (e.g. SVM) not yet implemented; requires compiling and curating a sufficient number of good and bad representative peaks for training samples
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository provided

## Evidence

- [abstract] MCQ index unsuited for high background / single peak: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Global zigzag index robustness: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] Outperforming peak quality metrics: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [abstract] Combined metrics efficiency: "Furthermore, using several peak quality metrics in combination is more efficient than individual metrics in peak quality evaluation."
- [methods] Four-module processing pipeline: "we developed our own data processing program in Matlab, which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection,"
- [methods] Three comparator tools: "Three open-source tools, XCMS [5], MZmine [11], and MAVEN [7], were employed."
- [methods] Ground truth peak definition: "a peak is considered a ground truth peak if it can be detected by multiple tools"
- [discussion] Dataset and Matlab scripts availability: "The LC/MS profile dataset and Matlab scripts are available upon request."
