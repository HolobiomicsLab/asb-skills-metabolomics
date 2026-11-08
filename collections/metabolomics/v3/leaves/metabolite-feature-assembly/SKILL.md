---
name: metabolite-feature-assembly
description: Use when raw LC-MS metabolomics measurements (e.g. mzML from human plasma
  spike-in experiments) must be converted into per-analyte feature intensities for
  label-free quantification, and no feature table exists yet.
when_to_use_negative:
- Input is already a detected/linked feature table — use feature linking or statistics
  tools (e.g. FeatureLinkerUnlabeledQT) instead of re-running feature detection.
- The goal is identification/database annotation rather than quantification — querying
  accurate masses against e.g. the Human Metabolome Database is a separate downstream
  step.
- Data are not small-molecule LC-MS maps (e.g. the algorithm's metabolite focus or
  non-LC separation without adaptation) — although applicable to lipidomics/peptidomics,
  verify the isotope-pattern SVM is appropriate for the analyte class.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: FeatureFinderMetabo
  role: 'Core tool: sensitive mass trace detection and hypothesis-driven assembly
    of traces into metabolite features with hybrid mass-deviation/isotope-profile
    scoring'
  repo: github.com/OpenMS/OpenMS
- name: OpenMS 1.10
  role: Open-source framework in which FeatureFinderMetabo is implemented and integrated
  repo: github.com/OpenMS/OpenMS
- name: TOPPView
  role: GUI for invoking FeatureFinderMetabo and visually inspecting detected features
  repo: github.com/OpenMS/OpenMS
- name: INIFileEditor
  role: Tool for modifying FeatureFinderMetabo parameter files
  repo: github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Downstream retention time correction applied to detected features before linking
  repo: github.com/OpenMS/OpenMS
- name: FeatureLinkerUnlabeledQT
  role: Downstream linking of coinciding features across multiple experiments after
    detection
  repo: github.com/OpenMS/OpenMS
- name: TOPPAS
  role: Graphical workflow editor to chain feature detection, RT correction, linking,
    and tabular export
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:71c37f9268b89c8940b638012090e5513aa45d91530376db464a38af5f2fb7b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - INIFileEditor
  - MapAlignmentPoseClustering
  - OpenMS
  - OpenMS 1.10
  - TOPPAS
  - TOPPView
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9629629629629629
n_claims: 27
---

# metabolite-feature-assembly

## Summary

Detects mass traces in LC-MS metabolomics data and assembles them into features (all signals from the same analyte species) using mass-to-charge spacing, co-elution, and an SVM-based isotope-pattern classifier, as implemented in FeatureFinderMetabo within OpenMS 1.10.

## When to use

Use when raw LC-MS metabolomics measurements (e.g. mzML from human plasma spike-in experiments) must be converted into per-analyte feature intensities for label-free quantification, and no feature table exists yet.

## When NOT to use

- Input is already a detected/linked feature table — use feature linking or statistics tools (e.g. FeatureLinkerUnlabeledQT) instead of re-running feature detection.
- The goal is identification/database annotation rather than quantification — querying accurate masses against e.g. the Human Metabolome Database is a separate downstream step.
- Data are not small-molecule LC-MS maps (e.g. the algorithm's metabolite focus or non-LC separation without adaptation) — although applicable to lipidomics/peptidomics, verify the isotope-pattern SVM is appropriate for the analyte class.

## Inputs

- LC-MS measurement files in OpenMS-supported formats (e.g. mzML) from repeated spike-in experiments in a human plasma background
- OpenMS INI parameter file (editable via INIFileEditor) for FeatureFinderMetabo settings

## Outputs

- Feature maps with assembled metabolite features (m/z, RT, charge, isotope pattern assignment)
- Feature intensities exported to a tabular text file for subsequent statistical analyses

## How to apply

Run FeatureFinderMetabo (OpenMS 1.10, callable standalone, from TOPPView, or via TOPPAS/KNIME workflows) on the LC-MS map to perform sensitive mass trace detection, then assemble traces into features based on mass-to-charge spacing, co-elution information, and an SVM-based classifier that identifies plausible metabolite isotope patterns. Score assembled features with a hybrid model combining mass deviation and isotope profile scoring so that only accurately assembled features are retained; parameters can be tuned through the INIFileEditor. Export the resulting feature intensities to a tabular text file and continue the quantification chain with retention time correction (MapAlignmentPoseClustering) and feature linking (FeatureLinkerUnlabeledQT). Expect roughly 2,020 ± 236 features per plasma measurement, of which ~351 are singly, ~12 doubly, and ~6 triply charged, and verify quantification linearity (Pearson R ≥ 0.98 between spike-in concentrations and feature intensities).

## Related tools

- **FeatureFinderMetabo** (Core tool: sensitive mass trace detection and hypothesis-driven assembly of traces into metabolite features with hybrid mass-deviation/isotope-profile scoring) — github.com/OpenMS/OpenMS
- **OpenMS 1.10** (Open-source framework in which FeatureFinderMetabo is implemented and integrated) — github.com/OpenMS/OpenMS
- **TOPPView** (GUI for invoking FeatureFinderMetabo and visually inspecting detected features) — github.com/OpenMS/OpenMS
- **INIFileEditor** (Tool for modifying FeatureFinderMetabo parameter files) — github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Downstream retention time correction applied to detected features before linking) — github.com/OpenMS/OpenMS
- **FeatureLinkerUnlabeledQT** (Downstream linking of coinciding features across multiple experiments after detection) — github.com/OpenMS/OpenMS
- **TOPPAS** (Graphical workflow editor to chain feature detection, RT correction, linking, and tabular export) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in plasma_spikein.mzML -out features.featureXML -ini ffm.ini; FeatureLinkerUnlabeledQT -in *.featureXML -out linked.featureXML; TextExporter -in linked.featureXML -out intensities.tsv
```

## Evaluation signals

- On simulated data matched to experimental conditions, precision ≈ 97% and recall ≈ 96% — both above the 95% threshold claimed for determining feature intensities.
- Pearson correlation between spike-in compound concentrations and detected feature intensities reaches R ≥ 0.98 for all standard compounds.
- Feature counts per plasma measurement fall near the reported 2,020 ± 236 (≈351 singly, ~12 doubly, ~6 triply charged); a set of 1,168 features should be reproducible in ≥50% of measurements.
- Output tabular file contains feature intensity columns compatible with downstream statistical analysis after MapAlignmentPoseClustering and FeatureLinkerUnlabeledQT.
- The benchmarked pipeline should outperform XCMS/CAMERA baselines in precision and recall on the same data.

## Limitations

- Signals at the lowest spike-in levels may be undetectable: no signals for the first six concentrations of propionyl-l-carnitine-d3 were found by the algorithm or visual inspection.
- The 'above 95% on typical data' sensitivity/specificity claim lacks an explicit definition of 'typical data' and exact cutoff values.
- No explicit description of the statistical learning method details, baseline comparison configurations, or quantitative spike-in details is provided in parts of the text; benchmark data must be loaded from the article's supplemental measurement files.
- Roughly 47% of detected features (≈1,066 ± 36 reproducible of ~2,020) are not reproducible in at least 50% of measurements and should not be treated as reliable quantification targets.

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "which are then assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [discussion] Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features.: "Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features."
- [results] The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R≥ 0.98 for all standard compounds): "The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R≥ 0.98 for all standard compounds)"
- [results] the algorithm detected on average 2,020 ± 236 features per measurement, of which about 351 were singly, 12 doubly, and 6 triply charged: "the algorithm detected on average 2,020 ± 236 features per measurement, of which about 351 were singly, 12 doubly, and 6 triply charged"
- [results] OpenMS offers the functionality to build custom processing and analysis pipelines through the graphical workflow editor TOPPAS: "OpenMS offers the functionality to build custom processing and analysis pipelines through the graphical workflow editor TOPPAS"
- [readme] OpenMS is an open-source software C++ library for LC-MS data management and analyses.: "is an open-source software C++ library for LC-MS data management and
analyses"
