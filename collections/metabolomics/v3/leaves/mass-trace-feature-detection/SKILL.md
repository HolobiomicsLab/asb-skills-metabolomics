---
name: mass-trace-feature-detection
description: Use when raw LC-MS metabolomics measurements (e.g. mzML) must be converted
  into feature-level intensities, especially when isotope-pattern-aware feature detection
  is needed before linking, alignment, or statistical analysis.
when_to_use_negative:
- Input is already a feature table or peak-picked feature list — the skill operates
  on raw LC-MS mass traces, not on existing quantification matrices.
- The analytes are intact proteins or another class outside the small-molecule scope
  (though the algorithm also applies to lipidomics and peptidomics, it is not built
  for large biomolecule deconvolution).
- Data are not chromatography-based (e.g. direct-infusion MS without co-elution information),
  since feature assembly relies on co-eluting mass traces.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FeatureFinderMetabo
  role: Core tool that detects mass traces and assembles them into features with SVM-based
    isotope-pattern classification
  repo: github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Retention-time drift correction across replicate runs before feature linking
  repo: github.com/OpenMS/OpenMS
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across multiple experiments into consensus features
  repo: github.com/OpenMS/OpenMS
- name: TOPPView
  role: GUI for running the tool chain and visually inspecting detected features
  repo: github.com/OpenMS/OpenMS
- name: INIFileEditor
  role: Edits FeatureFinderMetabo parameter files within the workflow
- name: TOPPAS
  role: Graphical workflow editor used to assemble the automated detection–linking–export
    pipeline
  repo: github.com/OpenMS/OpenMS
- name: KNIME
  role: Platform in which the full automated feature-detection/linking workflow was
    implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:c55e13525a52e5e0017198cb05326d454cd5bb9e41269f2a3c15e975cce51f48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - INIFileEditor
  - KNIME
  - MapAlignmentPoseClustering
  - OpenMS
  - TOPPAS
  - TOPPView
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# mass-trace-feature-detection

## Summary

Detects mass traces in LC-MS metabolomics data and assembles them into isotope-resolved features using OpenMS FeatureFinderMetabo, the basis for reliable label-free quantification.

## When to use

Use when raw LC-MS metabolomics measurements (e.g. mzML) must be converted into feature-level intensities, especially when isotope-pattern-aware feature detection is needed before linking, alignment, or statistical analysis.

## When NOT to use

- Input is already a feature table or peak-picked feature list — the skill operates on raw LC-MS mass traces, not on existing quantification matrices.
- The analytes are intact proteins or another class outside the small-molecule scope (though the algorithm also applies to lipidomics and peptidomics, it is not built for large biomolecule deconvolution).
- Data are not chromatography-based (e.g. direct-infusion MS without co-elution information), since feature assembly relies on co-eluting mass traces.

## Inputs

- mzML LC-MS measurement files (e.g. human plasma spike-in series acquired on a Waters Acquity/Synapt HDMS platform)
- OpenMS INI parameter files (editable via INIFileEditor)

## Outputs

- Per-measurement feature maps (detected features with mass traces, charge, isotope pattern, and intensity)
- Tabular text file of linked feature intensities exported via TextExporter for statistical analysis

## How to apply

Run FeatureFinderMetabo (OpenMS 1.10 or later) on each mzML measurement to detect mass traces and assemble them into features based on mass-to-charge spacing, co-elution information, and an SVM-based classifier that identifies potential metabolite isotope patterns. The scoring combines mass deviation and isotope-profile terms, so keep the SVM metabolite-pattern filter enabled for small-molecule data. Adjust parameters through the INIFileEditor or a TOPPAS/KNIME workflow rather than ad-hoc scripts, since the tool is designed to run fully automated across measurement batches. Downstream, feed the detected features into MapAlignmentPoseClustering for retention-time correction and FeatureLinkerUnlabeledQT for linking coinciding features across runs, then export intensities to a tabular text file via TextExporter. Expect roughly 2,020 ± 236 features per plasma measurement, of which about 351 are singly, 12 doubly, and 6 triply charged; large deviations flag parameter or data problems.

## Related tools

- **FeatureFinderMetabo** (Core tool that detects mass traces and assembles them into features with SVM-based isotope-pattern classification) — github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Retention-time drift correction across replicate runs before feature linking) — github.com/OpenMS/OpenMS
- **FeatureLinkerUnlabeledQT** (Links coinciding features across multiple experiments into consensus features) — github.com/OpenMS/OpenMS
- **TOPPView** (GUI for running the tool chain and visually inspecting detected features) — github.com/OpenMS/OpenMS
- **INIFileEditor** (Edits FeatureFinderMetabo parameter files within the workflow)
- **TOPPAS** (Graphical workflow editor used to assemble the automated detection–linking–export pipeline) — github.com/OpenMS/OpenMS
- **KNIME** (Platform in which the full automated feature-detection/linking workflow was implemented)

## Examples

```
FeatureFinderMetabo -in plasma_replicate_01.mzML -out features_01.featureXML -ini featurefindermetabo.ini
```

## Evaluation signals

- Feature counts per measurement should fall in the reported range (2,020 ± 236 total; ~1,066 ± 36 reproducible in ≥50% of measurements).
- Charge-state distribution sanity check: about 351 singly, 12 doubly, and 6 triply charged features per measurement.
- For spike-in standards, Pearson correlation between concentration and feature intensity should reach R ≥ 0.98.
- Technical-replicate feature intensities should vary little across triplicate runs after alignment and linking.
- Low-abundance levels may legitimately yield no signal (e.g. first six propionyl-l-carnitine-d3 concentrations); flag rather than force detection.

## Limitations

- Sensitivity claim of >95% is stated only for 'typical data' without explicit thresholds, parameter values, or definition of typical data.
- Some compounds can be below the detection limit — no signals were detectable for the first six concentrations of propionyl-l-carnitine-d3, even by visual inspection.
- Feature detection quality depends on parameter tuning (INI files); the paper does not report optimal parameter values, and reproducibility was demonstrated on OpenMS 1.10.
- Direct comparison baselines (XCMS/CAMERA) are described only at benchmark level without detailed comparison settings.

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [results] the algorithm detected on average 2,020 ± 236 features per measurement, of which about 351 were singly, 12 doubly, and 6 triply charged: "the algorithm detected on average 2,020 ± 236 features per measurement, of which about 351 were singly, 12 doubly, and 6 triply charged"
- [discussion] Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features.: "Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features."
- [readme] Over 150+ individual analysis tools (TOPP Tools), covering most MS and LC-MS data processing and mining tasks: [evidence span withheld — anchor status: external_doc]
