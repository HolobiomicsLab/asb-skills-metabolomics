---
name: synthetic-lc-ms-data-simulation
description: Use when you need a ground-truth-labeled LC-MS metabolite dataset to
  quantify precision/recall of a feature-detection algorithm under varied detector
  noise, mass error (10 or 40 ppm), or distorted elution profiles.
when_to_use_negative:
- Input is real experimental LC-MS data with no ground-truth feature annotations —
  matching-based precision/recall cannot be computed exactly
- The goal is downstream label-free quantification (retention-time correction, feature
  linking, export to tabular text) rather than detection-algorithm benchmarking
- The analytes are outside the article's scope of small molecules such as metabolites
  in complex backgrounds (e.g. human plasma spike-in linearity studies)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MSSimulator
  role: Generates the synthetic plant-metabolite LC-MS maps under Table S5 noise configurations
    and simultaneously yields the ground-truth feature lists
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: Runs feature detection on each simulated map across the parameter sweep so
    detected features can be matched against the simulator ground truth
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS 1.10
  role: Framework in which FeatureFinderMetabo is integrated and invoked for the benchmark
    runs
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:522c1d23fa2a518a8fa7b2705e372a74e416625bbd47a23a85e21a27a3acea89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - MSSimulator
  - OpenMS
  - OpenMS 1.10
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Synthetic LC-MS data simulation

## Summary

Generate synthetic plant-metabolite LC-MS maps with MSSimulator (supplemental Table S5 configurations: 500 metabolites, 100–1000 m/z, 25 min gradient) whose simulations also yield ground-truth feature lists, enabling recall/precision/F-score benchmarking of feature detection against known truth under controlled noise.

## When to use

Use when you need a ground-truth-labeled LC-MS metabolite dataset to quantify precision/recall of a feature-detection algorithm under varied detector noise, mass error (10 or 40 ppm), or distorted elution profiles.

## When NOT to use

- Input is real experimental LC-MS data with no ground-truth feature annotations — matching-based precision/recall cannot be computed exactly
- The goal is downstream label-free quantification (retention-time correction, feature linking, export to tabular text) rather than detection-algorithm benchmarking
- The analytes are outside the article's scope of small molecules such as metabolites in complex backgrounds (e.g. human plasma spike-in linearity studies)

## Inputs

- MSSimulator configuration file encoding supplemental Table S5 parameters (500 metabolites, 100–1000 m/z range, 25 min gradient, sigma_det detector-noise sigma, sigma_m/z = 10 and 40 ppm, ideal vs heavily distorted elution profiles)
- Feature-finder parameter sets for the FeatureFinderMetabo sweep

## Outputs

- Simulated synthetic plant-metabolite LC-MS maps per noise regime
- Ground-truth feature lists produced by MSSimulator
- Table of recall, precision, and F-score per simulator/feature-finder parameter setting
- Rendered benchmark figure (benchmark_scores.png)

## How to apply

Configure MSSimulator with the supplemental Table S5 settings — 500 metabolites spanning 100–1000 m/z on a 25 min gradient — sweeping detector noise (sigma_det), mass error (sigma_m/z = 10 and 40 ppm), and elution-profile distortion (ideal vs heavily distorted); the simulator emits both the synthetic map and its ground-truth feature list. Run FeatureFinderMetabo (OpenMS 1.10) on each simulated map across the parameter sweep of simulator and feature-finder settings. Match detected features against the MSSimulator ground truth and compute recall, precision, and F-score per the article's F-score definition for each parameter setting and noise regime. Tabulate scores versus simulator/feature-finder parameters and render the table as a plotted figure (benchmark_scores.png). The rationale is that only simulated data with known ground truth permits exact matching of predicted vs true features, so precision/recall of FeatureFinderMetabo can be measured exactly rather than approximated as with experimental data.

## Related tools

- **MSSimulator** (Generates the synthetic plant-metabolite LC-MS maps under Table S5 noise configurations and simultaneously yields the ground-truth feature lists) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Runs feature detection on each simulated map across the parameter sweep so detected features can be matched against the simulator ground truth) — https://github.com/OpenMS/OpenMS
- **OpenMS 1.10** (Framework in which FeatureFinderMetabo is integrated and invoked for the benchmark runs) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -ini ffmetabo.ini -in simulated_map.mzML -out detected.featureXML
```

## Evaluation signals

- Reproduced precision and recall under the baseline simulation regime reach ~97% and ~96%, respectively
- F-scores stay between 0.97 and 0.9 under increased mass error and chromatographic distortion
- Every detected feature set is matched against the MSSimulator ground truth, so recall can be computed exactly (detections are either true or false positives with no ambiguity)
- Recall/precision/F-score values per noise series match the values reported in the article's benchmark figures/tables within reasonable numerical tolerance

## Limitations

- The article claims high sensitivity and specificity 'above 95% on typical data' but provides no explicit threshold/parameter values and does not define what constitutes 'typical data'
- Ground truth exists only in simulation, so absolute performance on experimental maps is not directly measurable by this benchmark
- Benchmark conclusions depend on the Table S5 simulation choices (500 metabolites, 100–1000 m/z, 25 min gradient); other regimes require re-tuning the sweep

## Evidence

- [supplementary] Simulate synthetic plant-metabolite LC-MS maps with MSSimulator using the supplemental Table S5 configurations (500 metabolites: [evidence span withheld — anchor status: not_found]
- [supplementary] Match detected features against the MSSimulator ground truth and compute recall, precision, and F-score: [evidence span withheld — anchor status: not_found]
- [abstract] Relative to state-of-the art algorithms, ours demonstrated increased precision and recall of the method.: "Relative to state-of-the art algorithms, ours demonstrated increased precision and recall of the method."
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [discussion] The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data).: "The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data)."
