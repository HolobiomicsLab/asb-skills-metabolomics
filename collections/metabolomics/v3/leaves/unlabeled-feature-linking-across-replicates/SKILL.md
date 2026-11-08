---
name: unlabeled-feature-linking-across-replicates
description: Use when you have feature maps detected separately in multiple LC-MS
  runs (e.g., technical replicates or a dilution/spike-in series) and need a consensus
  feature map to extract per-run intensities for label-free quantification and reproducibility
  analysis.
when_to_use_negative:
- Input is already a linked consensus feature table or a tabular intensity matrix
  — re-linking would duplicate the workflow step
- Runs contain labeled spike-in/protein markers that support correspondence-based
  linking rather than unlabeled (QS-based) feature linking
- Only a single LC-MS measurement needs feature detection — link across replicates
  is unnecessary; use FeatureFinderMetabo alone
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding (unlabeled) features across the multiple experiments into
    a consensus map
  repo: github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Corrects retention-time drift across triplicate runs before linking so features
    correspond
  repo: github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: Upstream tool that detects mass traces and assembles them into the per-run
    features fed to the linker
  repo: github.com/OpenMS/OpenMS
- name: KNIME
  role: Workflow platform in which the detect → align → link → export pipeline was
    implemented for the 33 plasma runs
- name: TOPPAS
  role: OpenMS graphical workflow editor used to build the processing pipeline including
    the linking and export steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:e31a609996c0474d2108410b758f6782c8dfe84cadeb0d7035fe7fab2b30c3d7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# unlabeled-feature-linking-across-replicates

## Summary

Links unlabeled features detected independently in each LC-MS run into a consensus feature map, so that the same metabolite can be quantified consistently across replicates and concentration levels. In the source study this step (FeatureLinkerUnlabeledQT) underpins label-free quantification of 33 human plasma measurements from an 11-level spike-in series.

## When to use

Use when you have feature maps detected separately in multiple LC-MS runs (e.g., technical replicates or a dilution/spike-in series) and need a consensus feature map to extract per-run intensities for label-free quantification and reproducibility analysis.

## When NOT to use

- Input is already a linked consensus feature table or a tabular intensity matrix — re-linking would duplicate the workflow step
- Runs contain labeled spike-in/protein markers that support correspondence-based linking rather than unlabeled (QS-based) feature linking
- Only a single LC-MS measurement needs feature detection — link across replicates is unnecessary; use FeatureFinderMetabo alone

## Inputs

- Per-run feature maps from FeatureFinderMetabo (e.g., featureXML) for all measurements (33 mzML-derived runs: 11 concentration levels from 10 mg/l to 0.01 mg/l plus non-spiked control, triplicate, Waters Acquity/Synapt HDMS)
- Retention-time-corrected feature maps (post MapAlignmentPoseClustering)

## Outputs

- Consensus (linked) feature map assigning coinciding features across all 33 measurements
- Tabular text file of linked feature intensities per run for statistical analysis
- Reproducibility set of features present in >= 50% of measurements and per-standard intensity-vs-concentration series

## How to apply

Run feature detection per measurement first (FeatureFinderMetabo in OpenMS 1.10, which assembles mass traces into features via m/z spacing, co-elution, and an SVM-based isotope-pattern classifier), then correct retention-time drift between runs with MapAlignmentPoseClustering — alignment must precede linking so coinciding features correspond. Then apply FeatureLinkerUnlabeledQT across all measurements to link coinciding features from multiple experiments into a consensus map. Export the linked feature intensities to a tabular text file (e.g., via the TextExporter step in a TOPPAS or KNIME workflow) for downstream statistics. For the plasma spike-in validation, extract the linked intensities of each of the seven spiked standards and correlate intensity against spike-in concentration across the detectable levels, and quantify reproducibility by counting features present in at least 50% of the 33 measurements.

## Related tools

- **FeatureLinkerUnlabeledQT** (Links coinciding (unlabeled) features across the multiple experiments into a consensus map) — github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Corrects retention-time drift across triplicate runs before linking so features correspond) — github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Upstream tool that detects mass traces and assembles them into the per-run features fed to the linker) — github.com/OpenMS/OpenMS
- **KNIME** (Workflow platform in which the detect → align → link → export pipeline was implemented for the 33 plasma runs)
- **TOPPAS** (OpenMS graphical workflow editor used to build the processing pipeline including the linking and export steps)

## Examples

```
FeatureLinkerUnlabeledQT -in aligned_*.featureXML -out consensus.featureXML
```

## Evaluation signals

- The consensus map should cover on the order of 2,020 ± 236 detected features per measurement (about 351 singly, 12 doubly, 6 triply charged) in this plasma data set
- A reproducible subset of about 1,168 features present in at least 50% of all measurements, with about 1,066 ± 36 reproducible features per run (~53% of the average)
- For each of the seven spiked standards, Pearson correlation between spike-in concentration and linked feature intensity should reach R >= 0.98 across detectable levels
- Known failure case to verify detection limits: no signals for the first six (lowest) concentrations of propionyl-l-carnitine-d3 should be tolerated as non-detections
- Exported tabular intensities must contain one intensity column per run per linked consensus feature, traceable back to the 33 mzML measurements

## Limitations

- Low-abundance analytes can be entirely missed: no signals were detected for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or by visual inspection
- Only ~53% of detected features are reproducible in >= 50% of measurements, so a large fraction of linked features may not be usable for consistent quantification
- The published validation used OpenMS 1.10 and its specific FeatureFinderMetabo parameters; the paper does not report explicit threshold/parameter values or a code/data availability statement for reproducing the exact setup
- The study evaluated metabolites/small molecules; applicability is claimed broadly (lipidomics, peptidomics, other separations) but was validated here on human plasma LC-MS data

## Evidence

- [results] FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments: "FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments"
- [results] Tool chain description: "This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file"
- [results] Reproducible feature set: "We found a set of 1,168 features to be reproducible in at least 50% of all measurements"
- [results] RT correction prerequisite: "such as MapAlignmentPoseClustering for retention time correction"
- [readme] Workflow engine integration: [evidence span withheld — anchor status: external_doc]
