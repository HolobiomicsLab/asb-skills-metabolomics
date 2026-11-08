---
name: synthetic-chromatogram-generation
description: Use when validating or benchmarking LC/MS EIC or peak quality metrics
  (e.g., MCQ index, zigzag index, sharpness, SNR, TPASR) and you need reproducible
  synthetic chromatograms with known profile types as test fixtures.
when_to_use_negative:
- The input is a real LC/MS profile dataset (e.g., from the Agilent HPLC / Q-TOF Premier
  setup) whose genuine noise and baseline should be preserved rather than simulated
- The goal is peak detection (XCMS 'MatchedFilter'/'centWave', MZmine 'centroidPicker',
  MAVEN ML-based filtering) rather than validating quality-evaluation metrics
- Ground-truth peaks are needed for pipeline comparison — the paper defines ground
  truth as peaks detected by at least two of XCMS, MZmine, and MAVEN, which requires
  real data, not synthetic profiles
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0092
tools:
- name: Matlab
  role: Language in which the paper's four-module data processing program and the
    metric functions (MCQ index, zigzag index, sharpness, etc.) are implemented; the
    synthetic battery is generated and scored in Matlab
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:758d05fe5b47c1bd8ed651d68a16672a7319d2209b316acb25c1392051e8305c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# synthetic-chromatogram-generation

## Summary

Generates deterministic synthetic EIC and chromatographic peak profiles (smooth Gaussian-like, high-background, single-peak, and zigzag shapes with controlled SNR) so that LC/MS quality metrics such as the MCQ index, global/local zigzag index, sharpness, Gaussian similarity, SNR, peak significance level, and TPASR can be validated against known ground truth.

## When to use

Use when validating or benchmarking LC/MS EIC or peak quality metrics (e.g., MCQ index, zigzag index, sharpness, SNR, TPASR) and you need reproducible synthetic chromatograms with known profile types as test fixtures.

## When NOT to use

- The input is a real LC/MS profile dataset (e.g., from the Agilent HPLC / Q-TOF Premier setup) whose genuine noise and baseline should be preserved rather than simulated
- The goal is peak detection (XCMS 'MatchedFilter'/'centWave', MZmine 'centroidPicker', MAVEN ML-based filtering) rather than validating quality-evaluation metrics
- Ground-truth peaks are needed for pipeline comparison — the paper defines ground truth as peaks detected by at least two of XCMS, MZmine, and MAVEN, which requires real data, not synthetic profiles

## Inputs

- Fixed parameter set / random seed for the synthetic EIC battery (three profile types: normal, high-background, single-peak)
- Fixed random seed and specification for the synthetic peak set (apex index, left/right boundaries, SNR range, smooth vs. zigzag shape labels)

## Outputs

- Deterministic synthetic EIC profiles (intensity vectors) tagged with EIC_ID and EIC type
- Deterministic synthetic chromatographic peak profiles (intensity vectors) with apex index and left/right boundaries
- Per-case metric tables such as mcq_index_values.csv (EIC_ID, EIC type, MCQ_index) and peak_quality_metrics.csv (per-peak sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local zigzag index)

## How to apply

Build a deterministic synthetic EIC battery with three profile types — normal, high-background, and single-peak — using fixed parameters so every metric value is reproducible; this battery directly targets the paper's finding that the MCQ index (bounded 0–1.0) fails on high-background and single-peak EICs while the global zigzag index remains robust. For peak-level metrics, generate a deterministic synthetic peak set with a fixed random seed, each peak an intensity vector with a defined apex index and left/right boundaries, spanning smooth Gaussian-like and zigzag-shaped peaks across a range of SNR values and chromatogram locations. Then run each metric function (e.g., MCQ = similarity to smoothed version combined with similarity to mean-subtracted version; local zigzag index = normalized sum of squared second differences scaled by N × EPI², dynamic range 0–4.0) over every synthetic case and record one value per EIC_ID or peak ID. Validate outputs against expected directional behavior per profile type before using the generator for any downstream comparison.

## Related tools

- **Matlab** (Language in which the paper's four-module data processing program and the metric functions (MCQ index, zigzag index, sharpness, etc.) are implemented; the synthetic battery is generated and scored in Matlab)

## Evaluation signals

- All MCQ values must lie in the bounded dynamic range [0, 1.0]
- The normal EIC must score high on the MCQ index, while high-background and single-peak EICs must produce relatively low MCQ indices, matching the paper's known failure mode of the MCQ metric
- Smooth Gaussian-like peaks must score high on sharpness, Gaussian similarity, SNR, and peak significance level, and low on TPASR and the zigzag index
- Zigzag-shaped peaks must receive markedly higher local zigzag index values than smooth peaks
- Re-running the generator with the same fixed parameters/seed must reproduce identical metric values (bit-wise or within floating-point tolerance)

## Limitations

- Synthetic profiles must use fixed parameters/seed to be reproducible; the article's own LC/MS profile dataset and Matlab scripts are available only upon request, with no public repository
- The generator validates metric behavior directionally (high/low per profile type) but does not by itself establish optimal cutoff thresholds, which the paper notes remains an unsolved challenge
- The MCQ-index battery exposes a known limitation: the MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak, so it should not be treated as a general EIC quality metric

## Evidence

- [methods] Generate the deterministic synthetic EIC battery with three profile types (normal, high-background, single-peak) using fixed parameters so values are reproducible.: [evidence span withheld — anchor status: not_found]
- [methods] Generate a deterministic synthetic peak set (fixed random seed) of chromatographic peak profiles, each represented by its intensity vector with a defined apex index: [evidence span withheld — anchor status: not_found]
- [abstract] Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak.: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks: "the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks"
- [discussion] The LC/MS profile dataset and Matlab scripts are available upon request.: "The LC/MS profile dataset and Matlab scripts are available upon request."
