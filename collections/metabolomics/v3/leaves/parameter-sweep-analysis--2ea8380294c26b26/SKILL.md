---
name: parameter-sweep-analysis
description: Use when you must quantify how a feature-detection algorithm's recall,
  precision, and F-score degrade as detector noise, mass error (10 and 40 ppm), or
  elution-profile distortion is swept on simulated LC-MS data with a known ground
  truth.
when_to_use_negative:
- 'Input is real experimental data without ground truth (e.g., the article''s human
  plasma samples): recall/precision against simulated truth cannot be computed; use
  spike-in linearity (Pearson R ≥ 0.98) instead.'
- The question is quantification accuracy of feature intensities rather than detection
  robustness across noise regimes — the concentration-vs-intensity correlation evaluation
  is the appropriate alternative.
- Only a single fixed-parameter run is needed — a sweep adds no information when no
  parameters are being varied.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: MSSimulator
  role: Simulates plant-metabolite LC-MS maps per noise regime (detector-noise sigma_det,
    mass error sigma_m/z = 10/40 ppm, ideal vs heavily distorted elution profiles)
    and yields the ground-truth feature lists used for matching
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: The detector under evaluation; run on each simulated map across the sweep
    to produce feature lists that are matched to ground truth and scored
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: Open-source framework (version 1.10 in the article) providing the TOPP tools
    and parameter handling (INIFileEditor) needed to execute the sweep
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:b3e716030129abd565ec22860f857420202f36d6b69e3045299735c802e8ba03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - MSSimulator
  - OpenMS
  - OpenMS 1.10
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# parameter-sweep-analysis

## Summary

Synthetic-benchmark parameter sweep for LC-MS feature detection: vary MSSimulator noise regimes and FeatureFinderMetabo settings on simulated maps with known ground truth, then score recall, precision, and F-score per setting to trace robustness curves.

## When to use

Use when you must quantify how a feature-detection algorithm's recall, precision, and F-score degrade as detector noise, mass error (10 and 40 ppm), or elution-profile distortion is swept on simulated LC-MS data with a known ground truth.

## When NOT to use

- Input is real experimental data without ground truth (e.g., the article's human plasma samples): recall/precision against simulated truth cannot be computed; use spike-in linearity (Pearson R ≥ 0.98) instead.
- The question is quantification accuracy of feature intensities rather than detection robustness across noise regimes — the concentration-vs-intensity correlation evaluation is the appropriate alternative.
- Only a single fixed-parameter run is needed — a sweep adds no information when no parameters are being varied.

## Inputs

- Synthetic plant-metabolite LC-MS maps (mzML) simulated with MSSimulator at each grid point (sigma_det, sigma_m/z = 10/40 ppm, ideal vs heavily distorted elution profiles)
- Ground-truth feature lists emitted by MSSimulator alongside each simulated map
- Parameter-grid specification from supplemental Table S5 (500 metabolites, 100–1000 m/z, 25 min gradient) plus FeatureFinderMetabo settings to sweep
- Detected feature lists output by FeatureFinderMetabo for each map

## Outputs

- Recall, precision, and F-score computed per parameter setting and noise regime against the MSSimulator ground truth
- Table of scores versus simulator/feature-finder parameters
- Plotted robustness figure saved as benchmark_scores.png

## How to apply

Generate synthetic plant-metabolite LC-MS maps with MSSimulator over the noise grid of supplemental Table S5 (500 metabolites, 100–1000 m/z, 25 min gradient; detector-noise sigma_det, mass error sigma_m/z = 10 and 40 ppm, ideal vs heavily distorted elution profiles); each simulation also yields its ground-truth feature list. Run FeatureFinderMetabo (OpenMS, version 1.10 in the article) on every simulated map, sweeping both simulator and feature-finder settings so each grid cell is scored. Match detected features against the MSSimulator ground truth per map and compute recall, precision, and the article's F-score definition for every parameter setting and noise regime, varying one noise axis at a time so degradation is attributable. Tabulate the scores versus simulator/feature-finder parameters and render the table as a plotted figure saved as benchmark_scores.png. Judge success when the reproduced curves match the article's benchmark values — precision ~97% and recall ~96%, with F-scores between 0.97 and 0.9 under increased mass error and chromatographic distortion — within reasonable numerical tolerance.

## Related tools

- **MSSimulator** (Simulates plant-metabolite LC-MS maps per noise regime (detector-noise sigma_det, mass error sigma_m/z = 10/40 ppm, ideal vs heavily distorted elution profiles) and yields the ground-truth feature lists used for matching) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (The detector under evaluation; run on each simulated map across the sweep to produce feature lists that are matched to ground truth and scored) — https://github.com/OpenMS/OpenMS
- **OpenMS** (Open-source framework (version 1.10 in the article) providing the TOPP tools and parameter handling (INIFileEditor) needed to execute the sweep) — https://github.com/OpenMS/OpenMS

## Evaluation signals

- Precision ≈ 0.97 and recall ≈ 0.96 on the baseline simulated plant-metabolite configuration, matching the article's reported benchmark
- F-scores remain between 0.97 and 0.9 as mass error and chromatographic distortion increase across the sweep
- Every combination of simulator and feature-finder parameters in the grid has a tabulated recall/precision/F-score row
- Reproduced curves in benchmark_scores.png match the article's benchmark figures/tables within reasonable numerical tolerance
- Each simulated map has a paired ground-truth feature list (500 metabolites per simulation), so every score is computable from matched truth vs detected features

## Limitations

- Scores are defined against synthetic ground truth only; the article never defines what constitutes 'typical data' for its >95% sensitivity/specificity claim, so tolerances are unverified on experimental maps.
- The sweep is calibrated to plant-metabolite simulation configurations (500 metabolites, 100–1000 m/z, 25 min gradient); behavior in complex backgrounds such as human plasma was validated separately via spike-in linearity, not by this noise sweep.
- Very low-abundance analytes can produce no detectable signals at all (no signals for the first six concentrations of propionyl-l-carnitine-d3), so recall floors depend on the simulated intensity range.

## Evidence

- [abstract] Relative to state-of-the art algorithms, ours demonstrated increased precision and recall of the method.: "Relative to state-of-the art algorithms, ours demonstrated increased precision and recall of the method."
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [discussion] The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data).: "The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data)."
- [readme] With pyOpenMS, OpenMS offers Python bindings to a large part of the OpenMS API: [evidence span withheld — anchor status: external_doc]
- [other] the algorithm achieved precision and recall of 97% and 96% respectively, and F-scores stayed between 0.97 and 0.9 under increased mass error and chromatographic distortion: [evidence span withheld — anchor status: not_found]
