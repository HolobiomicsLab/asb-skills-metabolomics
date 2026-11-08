---
name: xcms-camera-adduct-cluster-annotation
description: Use when benchmarking an LC-MS feature-detection method against XCMS/CAMERA
  on the same measurements, and you need XCMS+CAMERA output as a comparable feature
  map with isotope/adduct annotations resolved.
when_to_use_negative:
- Input is already an OpenMS featureXML feature map from FeatureFinderMetabo — no
  XCMS/CAMERA re-detection or conversion is needed
- The goal is de novo metabolite quantification for a new study rather than reproducing
  or extending the published XCMS+CAMERA versus OpenMS benchmark
- Data are not small-molecule LC-MS feature-detection benchmarks (e.g. proteomics
  peptide identification), where the metabolite-focused isotope-pattern SVM of FeatureFinderMetabo
  and CAMERA adduct rules do not apply
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: XCMS
  role: State-of-the-art feature detection on the benchmark LC-MS measurements; paired
    with CAMERA for annotation in the head-to-head comparison
- name: CAMERA
  role: Annotates isotope and adduct clusters on XCMS features; adduct clusters are
    resolved and non-annotated features treated as singletons before comparison
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across replicate measurements so relative precision
    and recall can be computed against ground truth
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: Supplies the featureXML format for the converted XCMS+CAMERA maps and FeatureFinderMetabo
    as the comparison method (version 1.10)
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:f1984b88b145961d57aa1538ccfb6c51a3ec8de0f2688705832a050c6e428c91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - CAMERA
  - FeatureLinkerUnlabeledQT
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# xcms-camera-adduct-cluster-annotation

## Summary

Runs XCMS feature detection followed by CAMERA isotope/adduct annotation and converts the result into OpenMS featureXML feature maps, so XCMS+CAMERA feature sets can be compared head-to-head with FeatureFinderMetabo on identical LC-MS benchmark data.

## When to use

Use when benchmarking an LC-MS feature-detection method against XCMS/CAMERA on the same measurements, and you need XCMS+CAMERA output as a comparable feature map with isotope/adduct annotations resolved.

## When NOT to use

- Input is already an OpenMS featureXML feature map from FeatureFinderMetabo — no XCMS/CAMERA re-detection or conversion is needed
- The goal is de novo metabolite quantification for a new study rather than reproducing or extending the published XCMS+CAMERA versus OpenMS benchmark
- Data are not small-molecule LC-MS feature-detection benchmarks (e.g. proteomics peptide identification), where the metabolite-focused isotope-pattern SVM of FeatureFinderMetabo and CAMERA adduct rules do not apply

## Inputs

- Synthetic benchmark LC-MS measurements with simulated 10 ppm and 40 ppm mass errors and known ground-truth features
- Human plasma benchmark LC-MS measurements
- XCMS version (Bioconductor 1.34) and CAMERA version (1.14) software environments

## Outputs

- CAMERA-annotated XCMS feature tables with isotope and adduct clusters resolved (non-annotated features as singletons)
- OpenMS featureXML feature maps with monoisotopic isotope values retained
- Linked feature maps across replicates from FeatureLinkerUnlabeledQT
- Relative precision/recall (and F-score) comparison table against FeatureFinderMetabo ground truth

## How to apply

Run XCMS (Bioconductor 1.34) feature detection on the synthetic and human plasma benchmark LC-MS measurements, then annotate isotope and adduct clusters with CAMERA (1.14), resolving CAMERA adduct clusters so each analyte's co-detected signals collapse to one feature. Treat non-annotated features as singleton features so they are not discarded from the comparison. Convert the XCMS+CAMERA output to OpenMS featureXML feature maps, retaining monoisotopic isotope values so feature intensities are directly comparable to those from FeatureFinderMetabo (OpenMS 1.10). Link coinciding features across replicate measurements with FeatureLinkerUnlabeledQT and compute relative precision and recall against the known ground-truth features on both synthetic (10 ppm and 40 ppm mass error) and plasma datasets. Matching the method's mass error parameters to the simulated mass errors is the key decision point: with parameters matched to 10 ppm (40 ppm), XCMS/CAMERA reached F-score 0.52 (0.75) versus 0.97 (0.95) for the presented algorithm, so a correctly applied run reproduces this ranking.

## Related tools

- **XCMS** (State-of-the-art feature detection on the benchmark LC-MS measurements; paired with CAMERA for annotation in the head-to-head comparison)
- **CAMERA** (Annotates isotope and adduct clusters on XCMS features; adduct clusters are resolved and non-annotated features treated as singletons before comparison)
- **FeatureLinkerUnlabeledQT** (Links coinciding features across replicate measurements so relative precision and recall can be computed against ground truth) — https://github.com/OpenMS/OpenMS
- **OpenMS** (Supplies the featureXML format for the converted XCMS+CAMERA maps and FeatureFinderMetabo as the comparison method (version 1.10)) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureLinkerUnlabeledQT -in xcms_cam mapa_1.featureXML mapa_2.featureXML -out linked.featureXML
```

## Evaluation signals

- The XCMS+CAMERA feature maps must carry monoisotopic isotope values so feature intensities are comparable to FeatureFinderMetabo output
- All CAMERA non-annotated features must be retained as singleton features, not dropped from the comparison
- The computed relative precision and recall must reproduce the reported benchmark: XCMS/CAMERA F-score 0.52 (0.75) versus 0.97 (0.95) for the presented algorithm at 10 ppm (40 ppm) matched mass error parameters
- Confirmed by expert review of the comparison table on the same synthetic and human plasma datasets

## Limitations

- The comparison depends on matching mass error parameters to the simulated errors (10 ppm / 40 ppm); unmatched parameters would not reproduce the published F-scores
- Converting XCMS+CAMERICA output requires careful retention of monoisotopic isotope values; losing them breaks intensity comparability with FeatureFinderMetabo
- Non-annotated features (no CAMERA adduct/isotope assignment) must be handled as singletons, which affects the precision/recall computation

## Evidence

- [results] we repeated the benchmarking studies with the state-of-the-art software packages XCMS and CAMERA: "we repeated the benchmarking studies with the state-of-the-art software packages XCMS and CAMERA"
- [results] FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments: "FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments"
- [results] OpenMS 1.10 as FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [discussion] Even in complex backgrounds (human plasma), we found excellent linearity: "Even in complex backgrounds (human plasma), we found excellent linearity of the quantification based on repeated spike-in experiments."
- [readme] OpenMS runs on all major operating systems and provides TOPP tools: [evidence span withheld — anchor status: external_doc]
