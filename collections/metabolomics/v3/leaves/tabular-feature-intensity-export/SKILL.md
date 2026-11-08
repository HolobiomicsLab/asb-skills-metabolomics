---
name: tabular-feature-intensity-export
description: Use when a completed OpenMS feature-linking step (FeatureLinkerUnlabeledQT)
  has produced aligned feature maps across replicate LC-MS runs and the intensities
  must be exported to a tabular text file for statistical analysis.
when_to_use_negative:
- Input is already a tabular feature-intensity matrix exported from the pipeline —
  re-exporting would only duplicate it.
- Raw centroided peak data without prior feature detection, RT alignment, and linking
  — the export step has no linked features to write out.
- The analysis targets MS/MS identification outputs (peptide or metabolite IDs) rather
  than quantitative feature intensities across samples.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0091
tools:
- name: OpenMS TextExporter
  role: Terminal step of the TOPPAS/KNIME workflow that writes linked feature intensities
    to a tabular text file for subsequent statistical analyses
- name: FeatureLinkerUnlabeledQT
  role: Produces the consensus/linked feature maps whose intensities are exported;
    must run immediately before the export
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: Upstream feature-detection step in OpenMS 1.10 that creates the features whose
    intensities end up in the exported table
  repo: https://github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Retention-time correction upstream of linking; ensures feature intensities
    are matched across replicate runs before export
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPAS
  role: Graphical workflow editor in which the detection → alignment → linking → export
    tool chain is assembled and executed
- name: KNIME
  role: Alternative platform for implementing the export workflow with information-mining
    steps after the tabular export
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:dd4ff9abec9327d731779dcdac50e83091d99311e01635f3a2880857aab511d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - KNIME
  - MapAlignmentPoseClustering
  - OpenMS
  - OpenMS TextExporter
  - TOPPAS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# tabular-feature-intensity-export

## Summary

Exports linked LC-MS feature intensities from the OpenMS label-free quantification pipeline (FeatureFinderMetabo → MapAlignmentPoseClustering → FeatureLinkerUnlabeledQT) into a tabular text file so downstream statistics (e.g. Pearson correlation against spike-in concentrations) can be run outside OpenMS.

## When to use

Use when a completed OpenMS feature-linking step (FeatureLinkerUnlabeledQT) has produced aligned feature maps across replicate LC-MS runs and the intensities must be exported to a tabular text file for statistical analysis.

## When NOT to use

- Input is already a tabular feature-intensity matrix exported from the pipeline — re-exporting would only duplicate it.
- Raw centroided peak data without prior feature detection, RT alignment, and linking — the export step has no linked features to write out.
- The analysis targets MS/MS identification outputs (peptide or metabolite IDs) rather than quantitative feature intensities across samples.

## Inputs

- Linked feature maps from FeatureLinkerUnlabeledQT covering all LC-MS measurements (e.g. mzML-derived featureXML/consensusXML from a spike-in series such as 33 human plasma runs)
- OpenMS TOPPAS/KNIME workflow configuration (INI file, editable via INIFileEditor)

## Outputs

- Tabular text file of feature intensities, one row per linked feature and one column per measurement, suitable for statistical analyses in R or other statistics software
- Per-compound intensity series extractable for linearity and reproducibility assessment

## How to apply

Assemble the OpenMS tool chain — feature detection (FeatureFinderMetabo), retention time correction (MapAlignmentPoseClustering), and feature linking (FeatureLinkerUnlabeledQT) — in TOPPAS or KNIME, with this export as the terminal step. Feed the linked feature maps (consensus features across all measurements, e.g. the 33 randomized triplicate plasma spike-in runs) into the export tool, which writes feature intensities to a tabular text file. The exported matrix should carry one intensity per linked feature per measurement, so each spike-in standard's intensity trace across concentration levels (10 mg/l down to 0.01 mg/l plus non-spiked control) can be extracted. Validate the export by re-computing the Pearson correlation between compound concentration and exported intensity, expecting R ≥ 0.98 for all detectable standard compounds, and by confirming the reproducible-feature counts (~1,168 features present in ≥ 50% of measurements; 1,066 ± 36 per run of ~2,020 ± 236 detected). Missing entries (e.g. the six lowest propionyl-l-carnitine-d3 levels) should be preserved as non-detected rather than zero-filled or imputed.

## Related tools

- **OpenMS TextExporter** (Terminal step of the TOPPAS/KNIME workflow that writes linked feature intensities to a tabular text file for subsequent statistical analyses)
- **FeatureLinkerUnlabeledQT** (Produces the consensus/linked feature maps whose intensities are exported; must run immediately before the export) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (Upstream feature-detection step in OpenMS 1.10 that creates the features whose intensities end up in the exported table) — https://github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Retention-time correction upstream of linking; ensures feature intensities are matched across replicate runs before export) — https://github.com/OpenMS/OpenMS
- **TOPPAS** (Graphical workflow editor in which the detection → alignment → linking → export tool chain is assembled and executed)
- **KNIME** (Alternative platform for implementing the export workflow with information-mining steps after the tabular export)

## Examples

```
FeatureLinkerUnlabeledQT -in *.featureXML -out consensus.featureXML && TextExporter -in consensus.featureXML -out intensities.tsv
```

## Evaluation signals

- Exported table contains intensity values for the reproducible-feature set: ~1,168 features present in ≥ 50% of the 33 measurements, averaging 1,066 ± 36 per run.
- Pearson correlation between spike-in concentrations (10 mg/l to 0.01 mg/l) and exported feature intensities is R ≥ 0.98 for all detectable standard compounds.
- Total detected features per run should average ~2,020 ± 236 before reproducibility filtering.
- Non-detected levels (e.g. the six lowest propionyl-l-carnitine-d3 concentrations) remain absent/empty in the table rather than being zero-filled.
- Row/column cardinality matches the FeatureLinkerUnlabeledQT output: one row per consensus feature, one column per measurement.

## Limitations

- The export is only as good as the upstream linking: features not linked across runs (e.g. low-abundance propionyl-l-carnitine-d3 at the six lowest concentrations) produce missing entries.
- It writes a simple tabular text file — no embedded metadata or statistical annotations; all statistics happen downstream.
- Exported intensities reflect the OpenMS 1.10-era FeatureFinderMetabo algorithm; reproducing the paper's numbers requires matching versions and parameters.
- Applicability is not limited to metabolites (lipidomics, peptidomics also work), but quantification quality depends on feature detection performance, not on the export step itself.

## Evidence

- [results] tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file: "This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file for subsequent statistical analyses"
- [results] export to a tabular text file for subsequent statistical analyses: "and the ability to export to a tabular text file for subsequent statistical analyses"
- [results] workflow implemented in KNIME: "we implemented this workflow in KNIME, a powerful platform for information mining"
- [results] TOPPAS workflow editor: "OpenMS offers the functionality to build custom processing and analysis pipelines through the graphical workflow editor TOPPAS"
- [results] OpenMS 1.10 FeatureFinderMetabo upstream: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [discussion] linked-feature intensity quantification validated by spike-in linearity: "Even in complex backgrounds (human plasma), we found excellent linearity of the quantification based on repeated spike-in experiments."
- [readme] OpenMS supports label-free quantitation protocols: [evidence span withheld — anchor status: external_doc]
- [readme] TOPPTools integrate into workflow engines including KNIME: [evidence span withheld — anchor status: external_doc]
