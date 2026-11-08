---
name: featurexml-format-conversion
description: Use when benchmarking or cross-tool comparison requires converting XCMS/CAMERA
  feature output into OpenMS featureXML maps with monoisotopic isotope values preserved,
  so FeatureLinkerUnlabeledQT can link coinciding features and precision/recall can
  be computed on identical ground truth.
when_to_use_negative:
- Input is already a native OpenMS featureXML feature map produced by FeatureFinderMetabo
  — no conversion or isotope-value reconciliation is needed.
- The goal is raw peak-level conversion (e.g. mzML/mzXML handling) rather than feature-level
  interoperability between XCMS/CAMERA and OpenMS.
- Comparing tools that already emit OpenMS-compatible featureXML directly, where a
  conversion step could alter intensities or isotope annotations unnecessarily.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: OpenMS
  role: Target framework and featureXML format owner into which external feature lists
    are converted
  repo: https://github.com/OpenMS/OpenMS
- name: XCMS
  role: Source feature-detection tool (Bioconductor 1.34) whose output is converted
    into OpenMS featureXML maps
- name: CAMERA
  role: Annotates isotope and adduct clusters (v1.14) before conversion; non-annotated
    features are kept as singletons
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across replicate measurements on the converted featureXML
    maps
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: OpenMS baseline (OpenMS 1.10) whose results on the same data are compared
    after conversion
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:a6f9f106aa695144f4ea56cec6621aa4756701d4e312e9cfe5f5086e5459f8b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - CAMERA
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - OpenMS
  - XCMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# featurexml-format-conversion

## Summary

Converts externally produced LC-MS feature lists (e.g. XCMS+CAMERA output) into OpenMS featureXML feature maps, retaining monoisotopic isotope values so that feature intensities are comparable across tools and can be linked and scored with the OpenMS pipeline.

## When to use

Use when benchmarking or cross-tool comparison requires converting XCMS/CAMERA feature output into OpenMS featureXML maps with monoisotopic isotope values preserved, so FeatureLinkerUnlabeledQT can link coinciding features and precision/recall can be computed on identical ground truth.

## When NOT to use

- Input is already a native OpenMS featureXML feature map produced by FeatureFinderMetabo — no conversion or isotope-value reconciliation is needed.
- The goal is raw peak-level conversion (e.g. mzML/mzXML handling) rather than feature-level interoperability between XCMS/CAMERA and OpenMS.
- Comparing tools that already emit OpenMS-compatible featureXML directly, where a conversion step could alter intensities or isotope annotations unnecessarily.

## Inputs

- XCMS feature detection output (Bioconductor 1.34) on synthetic and human plasma benchmark LC-MS measurements
- CAMERA 1.14 isotope/adduct cluster annotations, with adduct clusters resolved and non-annotated features treated as singletons
- Ground-truth feature lists for the benchmark datasets (for subsequent precision/recall computation)
- Replicate LC-MS measurement feature maps (mzML-derived)

## Outputs

- OpenMS featureXML feature maps containing the converted XCMS+CAMERA features with retained monoisotopic isotope values
- Consensus feature maps from FeatureLinkerUnlabeledQT linking coinciding features across replicates
- Relative precision and recall values (e.g. F-scores) tabulated/plotted against FeatureFinderMetabo results on the same data

## How to apply

Run XCMS (Bioconductor 1.34) feature detection on the synthetic and human plasma benchmark LC-MS measurements, then annotate isotope and adduct clusters with CAMERA (1.14), resolving CAMERA adduct clusters and treating non-annotated features as singleton features. Convert the XCMS+CAMERA output to OpenMS featureXML feature maps, explicitly retaining monoisotopic isotope values — this is the key rationale, since only comparable monoisotopic intensities allow a fair head-to-head against FeatureFinderMetabo. With the converted maps, link coinciding features across replicate measurements using FeatureLinkerUnlabeledQT, which OpenMS provides specifically 'for linking coinciding features from multiple experiments'. Finally compute relative precision and recall against the known ground-truth features; as a correctness anchor, the published benchmark gives XCMS/CAMERA an F-score of 0.52 at 10 ppm and 0.75 at 40 ppm matched mass error versus 0.97 and 0.95 for the OpenMS method.

## Related tools

- **OpenMS** (Target framework and featureXML format owner into which external feature lists are converted) — https://github.com/OpenMS/OpenMS
- **XCMS** (Source feature-detection tool (Bioconductor 1.34) whose output is converted into OpenMS featureXML maps)
- **CAMERA** (Annotates isotope and adduct clusters (v1.14) before conversion; non-annotated features are kept as singletons)
- **FeatureLinkerUnlabeledQT** (Links coinciding features across replicate measurements on the converted featureXML maps) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (OpenMS baseline (OpenMS 1.10) whose results on the same data are compared after conversion) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureLinkerUnlabeledQT -in:center xcms_cam_1.featureXML xcms_cam_2.featureXML xcms_cam_3.featureXML -out linked_consensus.featureXML
```

## Evaluation signals

- Converted featureXML maps retain monoisotopic isotope values so feature intensities are directly comparable between XCMS+CAMERA and OpenMS features.
- FeatureLinkerUnlabeledQT accepts the converted maps without schema errors and links coinciding features across replicate measurements.
- Computed relative precision and recall reproduce the reported benchmark: F-score 0.52 (0.75) for XCMS/CAMERA at matched 10 ppm (40 ppm) mass error vs 0.97 (0.95) for the presented algorithm.
- Expert review confirms the comparison table matches the published XCMS+CAMERA vs OpenMS benchmark on the same synthetic and human plasma datasets.
- CAMERA clusters are fully resolved, with every non-annotated feature accounted for as a singleton — no features silently dropped during conversion.

## Limitations

- The conversion is specific to benchmark reproducibility: features not annotated by CAMERA must be treated as singleton features, which affects downstream clustering and scoring.
- Only monoisotopic isotope values are retained for comparability, so charged-isomer or adduct-level intensity information from CAMERA clusters is not carried through.
- The head-to-head comparison depends on matched mass error parameters (10 ppm and 40 ppm); without this matching, F-scores are not comparable to the published 0.52/0.75 vs 0.97/0.95 values.

## Evidence

- [methods] Convert the XCMS+CAMERA output to OpenMS featureXML feature maps, retaining monoisotopic isotope values so feature intensities are comparable.: [evidence span withheld — anchor status: not_found]
- [results] With mass error parameters matched to the simulated 10 ppm (40 ppm) error, XCMS/CAMERA reached an F-score of 0.52 (0.75) while the presented algorithm achieved 0.97 (0.95).: [evidence span withheld — anchor status: not_found]
- [results] we repeated the benchmarking studies with the state-of-the-art software packages XCMS and CAMERA: "we repeated the benchmarking studies with the state-of-the-art software packages XCMS and CAMERA"
- [readme] Major community file formats supported (mzML, mzXML, mzIdentXML, pepXML, mzTab, etc.): [evidence span withheld — anchor status: external_doc]
- [results] FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments: "FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments"
