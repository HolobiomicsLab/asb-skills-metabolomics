---
name: featurexml-feature-map-serialization
description: Use when a hypothesis-driven feature-assembly stage has accepted a final
  list of features (mass traces grouped by m/z spacing, co-elution, and SVM isotope-pattern
  classification) and the results must be written out in featureXML format for downstream
  TOPP/OpenMS tools.
when_to_use_negative:
- The input is already a featureXML or tabular feature table — this skill only serializes
  freshly assembled features, not re-encodes existing ones
- Mass traces have not yet been detected or assembled into features (the stage-1 mass
  trace detection and hypothesis-scoring steps must run first)
- The goal is linking coinciding features across multiple experiments — that is handled
  by FeatureLinkerUnlabeledQT downstream, not by this serialization step
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FeatureFinderMetabo
  role: Detects mass traces, assembles them into features, and writes the accepted
    features as a featureXML feature map
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: Framework providing the featureXML file format, CTD parameter handling, and
    library that hosts FeatureFinderMetabo
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPView
  role: Graphical inspection of the written featureXML feature map to verify feature
    placement and charge assignments
  repo: https://github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:5f3ed4e838b3de97fb081c6c019cdf99505ba2e964aa1e5d4c35e7879d110dcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  - TOPPView
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# featurexml-feature-map-serialization

## Summary

Persist the assembled metabolite features produced by FeatureFinderMetabo into an OpenMS featureXML feature map, storing each accepted feature with its monoisotopic trace, intensities, charge estimate, and non-overlapping isotope hypothesis membership.

## When to use

Use when a hypothesis-driven feature-assembly stage has accepted a final list of features (mass traces grouped by m/z spacing, co-elution, and SVM isotope-pattern classification) and the results must be written out in featureXML format for downstream TOPP/OpenMS tools.

## When NOT to use

- The input is already a featureXML or tabular feature table — this skill only serializes freshly assembled features, not re-encodes existing ones
- Mass traces have not yet been detected or assembled into features (the stage-1 mass trace detection and hypothesis-scoring steps must run first)
- The goal is linking coinciding features across multiple experiments — that is handled by FeatureLinkerUnlabeledQT downstream, not by this serialization step

## Inputs

- Accepted feature list from the hypothesis-driven feature-assembly stage (features assembled from up to six isotopic mass traces, scored by m/z-distance and elution-correlation, filtered by the SVM isotope-abundance classifier)
- Stage-1 mass trace list with chromatographic apex positions and FWHM extents per trace
- LC-MS raw map metadata (mzML input context) needed to attach intensities and charge estimates

## Outputs

- featureXML feature map (OpenMS format) containing accepted features with intensities and charge estimates
- Optionally, a tabular text file export of the feature list for subsequent statistical analyses

## How to apply

After the feature-assembly stage has greedily transferred hypotheses in descending total-score order into the accepted list — guaranteeing that no mass trace occurs in more than one feature — serialize the result as an OpenMS feature map in featureXML format. Each feature carries the assembled mass traces with chromatographic apex positions and FWHM extents, the integrated feature intensity, and a charge estimate (the paper reports ~351 singly, ~12 doubly, and ~6 triply charged features out of an average 2,020 ± 236 features per measurement). Because featureXML is a PSI/OpenMS-native format with CTD-based parameter handling, the file can be read directly by subsequent TOPP tools (e.g. MapAlignmentPoseClustering for retention time correction, FeatureLinkerUnlabeledQT for linking coinciding features across experiments) or exported to a tabular text file for statistical analysis, and can be inspected in TOPPView. Validation of the serialization is implicit in the downstream tool chain: the map must parse as a valid OpenMS feature map whose entries reflect only non-overlapping, SVM-accepted hypotheses consistent with the reported >95% sensitivity and specificity.

## Related tools

- **FeatureFinderMetabo** (Detects mass traces, assembles them into features, and writes the accepted features as a featureXML feature map) — https://github.com/OpenMS/OpenMS
- **OpenMS** (Framework providing the featureXML file format, CTD parameter handling, and library that hosts FeatureFinderMetabo) — https://github.com/OpenMS/OpenMS
- **TOPPView** (Graphical inspection of the written featureXML feature map to verify feature placement and charge assignments) — https://github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in raw_lcms.mzML -out features.featureXML
```

## Evaluation signals

- The output file parses as a valid OpenMS featureXML map (loadable in TOPPView and by downstream TOPP tools such as MapAlignmentPoseClustering and FeatureLinkerUnlabeledQT)
- Every feature in the map stems from a non-overlapping, SVM-accepted isotope hypothesis — no mass trace occurs in more than one feature
- Feature intensities and charge estimates are populated; charge distribution should be dominated by singly charged features (paper reports ~351 singly, ~12 doubly, ~6 triply charged per measurement)
- Feature counts per measurement fall in the reported range (about 2,020 ± 236 features per measurement on comparable human plasma LC-MS data)
- Exported tabular text derived from the same map reproduces the feature intensities used in linearity validation (Pearson R ≥ 0.98 for standard compounds)

## Limitations

- The 95% sensitivity/specificity claim applies to 'typical data', which the paper leaves undefined; no explicit thresholds or parameter values are reported for that claim
- Low-abundance analytes can yield no output at all: for propionyl-l-carnitine-d3, no signals were detected for the first six concentrations, either by the algorithm or visual inspection
- Serialization inherits any errors from upstream assembly — hypotheses rejected by the SVM isotope-pattern classifier or skipped due to overlap with accepted features will be absent from the featureXML map

## Evidence

- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [results] This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file: "This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file"
- [results] or out of TOPPView, the OpenMS graphical user interface for processing and analyzing mass spectrometry data: "or out of TOPPView, the OpenMS graphical user interface for processing and analyzing mass spectrometry data"
- [other] the final features are stored in featureXML format: [evidence span withheld — anchor status: not_found]
- [readme] Major community file formats supported (mzML, mzXML, mzIdentXML, pepXML, mzTab, etc.): [evidence span withheld — anchor status: external_doc]
