---
name: precision-recall-benchmark-comparison
description: Use when comparing metabolite feature detection algorithms on data with
  known ground truth (e.g., simulated measurements) and you need matched-parameter
  precision/recall/F-score values across tools like XCMS, CAMERA, and FeatureFinderMetabo.
when_to_use_negative:
- The data are raw spectra with no ground-truth feature list — precision/recall cannot
  be computed without known features
- You are running a single algorithm and do not need a head-to-head comparison across
  tools
- The competitor tool's output cannot be converted into a common format (e.g., featureXML)
  with comparable intensity definitions, making scores non-comparable
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Baseline feature detection method whose output is benchmarked against the
    reference algorithm; run on the same benchmark measurements.
- name: CAMERA
  role: Isotope/adduct annotation used to resolve XCMS feature clusters; resolved
    adduct clusters are converted with non-annotated features as singletons before
    benchmarking.
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across replicate measurements for both the reference
    and competitor methods so relative precision/recall can be computed per feature
    group.
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS (FeatureFinderMetabo)
  role: Reference method whose precision/recall/F-score is compared against the XCMS+CAMERA
    baseline; provides featureXML output used for consistent scoring.
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:331cedfd838ff299fc8121242d1a83338b40c084b5936543cfb92f4fea854645
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - CAMERA
  - FeatureLinkerUnlabeledQT
  - OpenMS
  - OpenMS (FeatureFinderMetabo)
  - XCMS
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# precision-recall-benchmark-comparison

## Summary

A benchmarking protocol that compares LC-MS feature detection methods (e.g., FeatureFinderMetabo vs. XCMS+CAMERA) using relative precision, recall, and F-score against ground-truth features, converting heterogeneous tool outputs into a common featureXML format so scores are computed on identical feature definitions.

## When to use

Use when comparing metabolite feature detection algorithms on data with known ground truth (e.g., simulated measurements) and you need matched-parameter precision/recall/F-score values across tools like XCMS, CAMERA, and FeatureFinderMetabo.

## When NOT to use

- The data are raw spectra with no ground-truth feature list — precision/recall cannot be computed without known features
- You are running a single algorithm and do not need a head-to-head comparison across tools
- The competitor tool's output cannot be converted into a common format (e.g., featureXML) with comparable intensity definitions, making scores non-comparable

## Inputs

- Benchmark LC-MS measurements with known ground-truth features (e.g., simulated metabolite datasets with 10 ppm or 40 ppm mass errors, human plasma samples)
- XCMS feature detection output on the same measurements
- CAMERA isotope/adduct annotation output (version 1.14)
- OpenMS featureXML feature maps for each method
- Ground-truth feature list per measurement

## Outputs

- Converted featureXML feature maps for the competitor method (XCMS+CAMERA), with monoisotopic isotope values retained
- Linked feature sets across replicates per method (via FeatureLinkerUnlabeledQT)
- Relative precision and recall values per method against ground truth
- F-score per method per mass error setting (e.g., 0.52 vs 0.97 at 10 ppm; 0.75 vs 0.95 at 40 ppm)
- Comparison table/plot of method performance

## How to apply

Run each candidate algorithm on the same benchmark LC-MS measurements (synthetic data with known features plus human plasma). For XCMS (Bioconductor 1.34) followed by CAMERA (1.14), resolve CAMERA adduct clusters and treat non-annotated features as singleton features; convert the XCMS+CAMERA result into OpenMS featureXML feature maps, retaining monoisotopic isotope values so feature intensities are comparable to the FeatureFinderMetabo output. Match the competitor's mass error parameters to the simulated mass errors (e.g., 10 ppm or 40 ppm) so the comparison is parameter-fair. Link coinciding features across replicate measurements with FeatureLinkerUnlabeledQT, then compute relative precision and recall of each method against the known ground-truth features and derive F-scores. Validation is achieved when the computed values reproduce the reported benchmark (XCMS/CAMERA F-score 0.52 at 10 ppm and 0.75 at 40 ppm versus 0.97/0.95 for the OpenMS method).

## Related tools

- **XCMS** (Baseline feature detection method whose output is benchmarked against the reference algorithm; run on the same benchmark measurements.)
- **CAMERA** (Isotope/adduct annotation used to resolve XCMS feature clusters; resolved adduct clusters are converted with non-annotated features as singletons before benchmarking.)
- **FeatureLinkerUnlabeledQT** (Links coinciding features across replicate measurements for both the reference and competitor methods so relative precision/recall can be computed per feature group.) — https://github.com/OpenMS/OpenMS
- **OpenMS (FeatureFinderMetabo)** (Reference method whose precision/recall/F-score is compared against the XCMS+CAMERA baseline; provides featureXML output used for consistent scoring.) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureLinkerUnlabeledQT -in features_xcms.featureXML features_openms.featureXML -out linked.featureXML
```

## Evaluation signals

- Reproduced F-score values match the reported benchmark (XCMS/CAMERA 0.52 at 10 ppm, 0.75 at 40 ppm; reference method 0.97 and 0.95)
- Competitor mass error parameters were set to the simulated mass errors (10 ppm or 40 ppm) before scoring
- Converted XCMS+CAMERA featureXML maps retain monoisotopic isotope values, making intensities comparable to the OpenMS method
- Relative precision and recall are computed against the same ground-truth feature list for all methods
- Comparison was confirmed by expert review of the comparison table

## Limitations

- The benchmark depends on the quality and representativeness of the ground truth (simulated data with prescribed mass errors); results on real data without ground truth cannot be scored the same way.
- Converting XCMS+CAMERA output to OpenMS featureXML requires careful retention of monoisotopic isotope values; if intensities are not comparable, precision/recall values are not fair.
- The comparison is only parameter-fair when mass error settings are explicitly matched to the simulated mass errors (10 ppm / 40 ppm).

## Evidence

- [results] XCMS/CAMERA reached an F-score of 0.52 (0.75) while the presented algorithm achieved 0.97 (0.95) with matched mass error parameters.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step: run XCMS then CAMERA, resolve adduct clusters, treat non-annotated features as singletons.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step: convert XCMS+CAMERA output to featureXML, retaining monoisotopic isotope values for comparable intensities.: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step: link coinciding features across replicates with FeatureLinkerUnlabeledQT.: [evidence span withheld — anchor status: not_found]
- [methods] Validation: computed precision/recall must reproduce the reported benchmark on the same datasets.: [evidence span withheld — anchor status: not_found]
