---
name: cross-run-feature-linking
description: Use when multiple LC-MS runs (e.g. replicate measurements or concentration
  series in mzML/featureXML) must be matched feature-to-feature to build a quantification
  matrix; the article applies it after feature detection and retention time correction
  on human plasma data.
when_to_use_negative:
- Only a single LC-MS run is available — linking requires multiple maps and will yield
  nothing beyond the single input
- Features have not yet been detected or retention-time-corrected; linking raw peak
  maps or unaligned feature maps will produce spurious groups across runs
- A labelled (ground-truth-guided) linking strategy is needed — FeatureLinkerUnlabeledQT
  matches unlabeled maps by mass/RT similarity, not by known identities
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: FeatureLinkerUnlabeledQT
  role: 'Primary tool: links coinciding features from multiple experiments into consensus
    features for cross-run quantification'
- name: MapAlignmentPoseClustering
  role: 'Upstream prerequisite: corrects retention time across runs so that linking
    is performed on aligned coordinates'
- name: FeatureFinderMetabo
  role: Produces the per-run featureXML maps that serve as linking input
  repo: github.com/OpenMS/OpenMS
- name: TOPPAS
  role: Optional graphical workflow editor used to chain feature detection, alignment,
    and linking into a reproducible pipeline
- name: KNIME
  role: Alternative platform in which the full detection–alignment–linking workflow
    was implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:361b0585d21077fdd210a1818b017b80bbe2dd06e073d1132831a54eea13b3f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - KNIME
  - MapAlignmentPoseClustering
  - OpenMS
  - TOPPAS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Cross-run feature linking

## Summary

Link coinciding features across replicate or condition-wise LC-MS runs so that one feature ID carries a consistent intensity vector for downstream quantification and statistics, as used in the OpenMS label-free metabolomics pipeline with FeatureLinkerUnlabeledQT.

## When to use

Use when multiple LC-MS runs (e.g. replicate measurements or concentration series in mzML/featureXML) must be matched feature-to-feature to build a quantification matrix; the article applies it after feature detection and retention time correction on human plasma data.

## When NOT to use

- Only a single LC-MS run is available — linking requires multiple maps and will yield nothing beyond the single input
- Features have not yet been detected or retention-time-corrected; linking raw peak maps or unaligned feature maps will produce spurious groups across runs
- A labelled (ground-truth-guided) linking strategy is needed — FeatureLinkerUnlabeledQT matches unlabeled maps by mass/RT similarity, not by known identities

## Inputs

- Per-run feature maps in OpenMS featureXML format (produced by FeatureFinderMetabo)
- Retention-time-aligned feature maps (post MapAlignmentPoseClustering)
- Raw LC-MS runs in mzML (indirectly, via the upstream feature-detection step)

## Outputs

- Consensus feature map (featureXML) grouping corresponding features across all linked runs
- Feature-vs-run quantification matrix exportable to tabular text for downstream statistics

## How to apply

First detect features per run with FeatureFinderMetabo (OpenMS 1.10), then correct retention time drift with MapAlignmentPoseClustering so the same analyte elutes at comparable coordinates across runs. Run FeatureLinkerUnlabeledQT on the aligned feature maps to group coinciding features — the tool links features without requiring a reference file, matching on mass and aligned retention time. The output is a consensus feature map where each consensus feature aggregates the corresponding feature from every input map, ready for export to a tabular text file for statistical analysis. This linking step is what makes the reported reproducibility metrics (1,168 features reproducible in ≥50% of 33 plasma measurements) computable: a feature only counts as reproducible if it is linked across that fraction of runs.

## Related tools

- **FeatureLinkerUnlabeledQT** (Primary tool: links coinciding features from multiple experiments into consensus features for cross-run quantification)
- **MapAlignmentPoseClustering** (Upstream prerequisite: corrects retention time across runs so that linking is performed on aligned coordinates)
- **FeatureFinderMetabo** (Produces the per-run featureXML maps that serve as linking input) — github.com/OpenMS/OpenMS
- **TOPPAS** (Optional graphical workflow editor used to chain feature detection, alignment, and linking into a reproducible pipeline)
- **KNIME** (Alternative platform in which the full detection–alignment–linking workflow was implemented)

## Examples

```
FeatureLinkerUnlabeledQT -in run1.featureXML run2.featureXML run3.featureXML -out consensus.featureXML
```

## Evaluation signals

- Each consensus feature in the output map should contain one grouped feature from each of the input maps (or be marked missing), and the number of consensus features should be ≤ the minimum per-run feature count (~2,020 ± 236 features per plasma measurement).
- Cross-run reproducibility check: count consensus features present in ≥50% of the 33 human plasma runs — the article reports 1,168 such features, giving an expected order of magnitude for this dataset.
- Per-run coverage: roughly 1,066 ± 36 of the ~2,020 detected features per run should appear in linked consensus features (≈53%); a large deviation signals misalignment or wrong linking parameters.
- Validate on the simulated benchmark where ground-truth feature intensities are known: linear quantification after linking should give Pearson R ≥ 0.98 between true concentrations and linked feature intensities.
- The linked output must retain monoisotopic feature intensities (a requirement also stated when converting XCMS/CAMERA output to featureXML) so intensities are directly comparable across runs.

## Limitations

- Linking quality depends on prior retention-time correction; residual RT drift across runs causes the same analyte to appear in separate consensus features.
- Reproducible coverage is only ~53% of detected features (1,066 ± 36 of 2,020 ± 236) even in controlled plasma measurements — many features will remain unlinked or inconsistent across runs.
- The article's benchmark against XCMS+CAMERA required converting competitor output to featureXML and explicitly retaining monoisotopic isotope values to make intensities comparable — cross-tool linking comparisons are sensitive to such format conversions.
- Low-abundance analytes may not link at all: no signals were detected for the first six concentrations of propionyl-l-carnitine-d3, so they cannot participate in any cross-run consensus.
- The paper does not report version-specific parameter values for FeatureLinkerUnlabeledQT; users must tune mass and RT tolerances for their instrument's mass accuracy (the benchmark used 10 ppm and 40 ppm mass-error scenarios).

## Evidence

- [results] FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments: "FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments"
- [results] This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file: "This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file for subsequent statistical analyses"
- [results] We found a set of 1,168 features to be reproducible in at least 50% of all measurements: "We found a set of 1,168 features to be reproducible in at least 50% of all measurements"
- [results] OpenMS offers the functionality to build custom processing and analysis pipelines through the graphical workflow editor TOPPAS: "OpenMS offers the functionality to build custom processing and analysis pipelines through the graphical workflow editor TOPPAS"
- [results] such as MapAlignmentPoseClustering for retention time correction: "such as MapAlignmentPoseClustering for retention time correction"
