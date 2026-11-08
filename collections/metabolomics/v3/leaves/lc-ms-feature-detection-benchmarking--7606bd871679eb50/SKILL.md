---
name: lc-ms-feature-detection-benchmarking
description: Use when evaluating or comparing LC-MS metabolomics feature-detection
  tools on data with known ground truth, or when tuning FeatureFinderMetabo parameters
  under varying detector noise, mass error (e.g. 10 ppm vs 40 ppm), and distorted
  elution profiles.
when_to_use_negative:
- Input is already a validated feature table or quantification matrix — benchmarking
  requires raw LC-MS maps and a ground truth, not downstream results.
- The goal is biological interpretation of real samples (e.g. lipidomics or peptidomics
  identification) rather than algorithmic performance evaluation.
- Ground truth is unavailable (purely experimental data with no simulated reference)
  — precision/recall cannot be computed without known features.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: OpenMS
  role: Framework providing FeatureFinderMetabo, the feature-detection tool under
    evaluation; used for mass trace detection, feature assembly, and linking.
  repo: github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: Primary feature-detection algorithm benchmarked; run on each simulated map
    across a parameter sweep of the simulator and feature-finder settings.
  repo: github.com/OpenMS/OpenMS
- name: XCMS
  role: Competing feature-detection tool run on the same synthetic and human plasma
    benchmark measurements for head-to-head precision/recall comparison.
- name: CAMERA
  role: Annotates isotope and adduct clusters on XCMS output (version 1.14); non-annotated
    features are treated as singletons before comparison.
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across replicate measurements to enable reproducibility-based
    precision/recall computation.
  repo: github.com/OpenMS/OpenMS
- name: MSSimulator
  role: Generates synthetic plant-metabolite LC-MS maps and their ground-truth feature
    lists (supplemental Table S5 configurations).
- name: TOPPView
  role: OpenMS GUI for processing and analyzing mass spectrometry data, including
    modifying parameters via INIFileEditor.
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:1d271a0cdb76b82a222444732ade30812d3741a5ae782fb80a33507fc5e5131f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMERA
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - MSSimulator
  - OpenMS
  - OpenMS 1.10
  - TOPPView
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# lc-ms-feature-detection-benchmarking

## Summary

A benchmarking skill for evaluating LC-MS feature-detection algorithms by simulating metabolite maps with MSSimulator, running FeatureFinderMetabo (OpenMS) and competing tools (XCMS + CAMERA) on identical data, and scoring precision/recall/F-score against the simulated ground truth.

## When to use

Use when evaluating or comparing LC-MS metabolomics feature-detection tools on data with known ground truth, or when tuning FeatureFinderMetabo parameters under varying detector noise, mass error (e.g. 10 ppm vs 40 ppm), and distorted elution profiles.

## When NOT to use

- Input is already a validated feature table or quantification matrix — benchmarking requires raw LC-MS maps and a ground truth, not downstream results.
- The goal is biological interpretation of real samples (e.g. lipidomics or peptidomics identification) rather than algorithmic performance evaluation.
- Ground truth is unavailable (purely experimental data with no simulated reference) — precision/recall cannot be computed without known features.

## Inputs

- Synthetic LC-MS metabolite maps (simulated with MSSimulator, e.g. 500 metabolites, 100–1000 m/z, 25 min gradient)
- Ground-truth feature lists produced by MSSimulator
- Benchmark LC-MS measurements (e.g. human plasma, plant metabolite extracts)
- Feature-finder parameter settings (mass error tolerance, detector noise, elution-profile distortion level)

## Outputs

- Per-parameter precision, recall, and F-score table (FeatureFinderMetabo vs XCMS+CAMERA)
- Plotted benchmark figure (benchmark_scores.png) of scores vs simulator/feature-finder parameters
- OpenMS featureXML feature maps from each tool
- Comparison table of relative precision and recall for XCMS+CAMERA versus FeatureFinderMetabo

## How to apply

1. Simulate synthetic LC-MS metabolite maps with MSSimulator (e.g. 500 plant metabolites, 100–1000 m/z, 25 min gradient) across parameter sweeps covering detector noise sigma_det, mass error sigma_m/z = 10 and 40 ppm, and ideal vs heavily distorted elution profiles; MSSimulator also emits the ground-truth feature list. 2. Run FeatureFinderMetabo (OpenMS 1.10) on each simulated map and, for head-to-head comparison, run XCMS (Bioconductor 1.34) followed by CAMERA (1.14) to annotate isotope/adduct clusters, converting output to OpenMS featureXML so non-annotated features become singletons and monoisotopic isotope values are retained. 3. Link coinciding features across replicates with FeatureLinkerUnlabeledQT. 4. Match detected features against the MSSimulator ground truth and compute recall, precision, and F-score per the article's F-score definition for every simulator/feature-finder parameter setting, then tabulate scores versus noise regime and render benchmark_scores.png. Rationale: feature intensity comparability requires shared featureXML semantics, and matching errors against known simulated masses isolates detection quality from downstream quantification.

## Related tools

- **OpenMS** (Framework providing FeatureFinderMetabo, the feature-detection tool under evaluation; used for mass trace detection, feature assembly, and linking.) — github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Primary feature-detection algorithm benchmarked; run on each simulated map across a parameter sweep of the simulator and feature-finder settings.) — github.com/OpenMS/OpenMS
- **XCMS** (Competing feature-detection tool run on the same synthetic and human plasma benchmark measurements for head-to-head precision/recall comparison.)
- **CAMERA** (Annotates isotope and adduct clusters on XCMS output (version 1.14); non-annotated features are treated as singletons before comparison.)
- **FeatureLinkerUnlabeledQT** (Links coinciding features across replicate measurements to enable reproducibility-based precision/recall computation.) — github.com/OpenMS/OpenMS
- **MSSimulator** (Generates synthetic plant-metabolite LC-MS maps and their ground-truth feature lists (supplemental Table S5 configurations).)
- **TOPPView** (OpenMS GUI for processing and analyzing mass spectrometry data, including modifying parameters via INIFileEditor.) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in simulated_map.mzML -out detected_features.featureXML; XCMS+CAMERA output converted to featureXML and linked with FeatureLinkerUnlabeledQT, then scores tabulated vs simulator/feature-finder parameters and saved as benchmark_scores.png
```

## Evaluation signals

- Reproduced recall, precision, and F-score values under each noise series (sigma_det, sigma_m/z = 10 and 40 ppm, ideal vs heavily distorted elution profiles) match the article's benchmark figures/tables within reasonable numerical tolerance.
- XCMS+CAMERA F-scores reproduce the reported comparison: 0.52 (0.75) at 10 ppm (40 ppm) matched mass-error parameters, versus 0.97 (0.95) for FeatureFinderMetabo.
- The ground-truth feature list is correctly propagated from MSSimulator; detected features must be matched against it to compute recall, not merely counted.
- XCMS+CAMERA output converted to featureXML retains monoisotopic isotope values and treats non-annotated features as singletons so intensities are comparable across tools.
- All benchmark scores are tabulated against simulator and feature-finder parameter sweeps and rendered as benchmark_scores.png without missing cells.

## Limitations

- Quantitative results depend on the simulated configurations from supplemental Table S5; extrapolating to other metabolite counts, m/z ranges, or gradient lengths is unvalidated.
- The reported sensitivity and specificity above 95% applies only to 'typical data', which the article does not define.
- No explicit comparison against baselines beyond XCMS and CAMERA is provided, and CAMERA version (1.14) and XCMS Bioconductor version (1.34) were fixed; newer tool versions may shift results.
- In the case of the propionyl-l-carnitine-d3 standard, no signals could be detected for the first six concentrations, so low-abundance features may be absent from both the algorithm and visual inspection.
- The article provides no code or data availability statement for FeatureFinderMetabo beyond its inclusion in OpenMS 1.10, and no DOI or repository URL for the benchmarking pipeline itself.

## Evidence

- [results] On simulated plant-metabolite data configured to match experimental conditions, the algorithm achieved precision and recall of 97% and 96% respectively, and F-scores stayed between 0.97 and 0.9 under increased mass error and chromatographic distortion.: [evidence span withheld — anchor status: not_found]
- [results] With mass error parameters matched to the simulated 10 ppm (40 ppm) error, XCMS/CAMERA reached an F-score of 0.52 (0.75) while the presented algorithm achieved 0.97 (0.95).: [evidence span withheld — anchor status: not_found]
- [supplementary] Simulate synthetic plant-metabolite LC-MS maps with MSSimulator using the supplemental Table S5 configurations (500 metabolites, 100–1000 m/z, 25 min gradient, detector noise sigma_det, mass error sigma_m/z = 10 and 40 ppm, ideal vs heavily distorted elution profiles), which also yield the ground-truth feature lists.: [evidence span withheld — anchor status: not_found]
- [methods] Annotate isotope and adduct clusters with CAMERA (1.14), resolving CAMERA adduct clusters and treating non-annotated features as singleton features.: [evidence span withheld — anchor status: not_found]
- [readme] OpenMS is an open-source software C++ library for LC-MS data management and analyses. It offers an infrastructure for rapid development of mass spectrometry-related software.: [evidence span withheld — anchor status: not_found]
