---
name: metabolomics-json-data-serialization
description: Use when pipeline stages (Asari feature tables → khipu pre-annotation
  → MS1/MS2 annotation) must persist intermediate results in an interoperable, JSON-compatible
  data model such as empCpd.json or experiment.json.
when_to_use_negative:
- Input is already a fully annotated, pipeline-issued JSON artifact (e.g. an existing
  empCpd.json) that only needs downstream statistical analysis rather than re-serialization.
- The target downstream tool requires tabular formats rather than the JSON-compatible
  MetDataModel structures (pcpfm also emits standardized .txt feature tables for that
  case).
- The data model must deviate from the minimal MetDataModel core classes; internal
  structures are explicitly not meant to be final and interoperability depends on
  adhering to the core concepts.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetDataModel
  role: Provides the minimal, JSON-compatible core data models (Spectrum, Feature,
    Empirical compound, Experiment) that are serialized to JSON between pipeline steps.
  repo: https://github.com/shuzhao-li-lab/metDataModel
- name: pcpfm
  role: Python-centric pipeline that writes and consumes the JSON monikers (empCpd.json,
    experiment.json) across build_empCpds, map_ms2, and l2_annotate operations.
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python 3
  role: Implementation language in which the MetDataModel data structures and their
    JSON serialization are demonstrated.
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:009ddc6ccbe3980853424e589106194bb5bd590ef84ff199990b21d754acbdf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/metDataModel
  tools:
  - MatchMS
  - MetDataModel
  - MoNA
  - Python
  - Python 3
  - pcpfm
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# metabolomics-json-data-serialization

## Summary

Serializes metabolomics processing artifacts (empirical compounds, feature tables, experiments) into JSON using the MetDataModel data models, so that empCpds, annotations, and experiment state persist as chainable, interoperable JSON files between pcpfm pipeline steps.

## When to use

Use when pipeline stages (Asari feature tables → khipu pre-annotation → MS1/MS2 annotation) must persist intermediate results in an interoperable, JSON-compatible data model such as empCpd.json or experiment.json.

## When NOT to use

- Input is already a fully annotated, pipeline-issued JSON artifact (e.g. an existing empCpd.json) that only needs downstream statistical analysis rather than re-serialization.
- The target downstream tool requires tabular formats rather than the JSON-compatible MetDataModel structures (pcpfm also emits standardized .txt feature tables for that case).
- The data model must deviate from the minimal MetDataModel core classes; internal structures are explicitly not meant to be final and interoperability depends on adhering to the core concepts.

## Inputs

- Asari-derived feature table (preferred_Feature_table.tsv or full_feature_table.tsv from asari_results/export)
- Mapped empirical compound JSON (e.g. moniker 'MS2_mapped') from the map_ms2 step
- Reference MS2 spectral library matches (MoNA .msp or custom .msp) to attach as Level 2 annotations
- Experiment metadata (sample names, file paths, sample types such as BLANK/QC/STD, Batch fields)

## Outputs

- empCpd.json in the experiment's annotations directory (empirical compounds with grouped features and adduct/isotope pre-annotations)
- Updated empCpd JSON monikers after each annotation step (e.g. 'MS2_MS1_annotated')
- experiment.json describing the full experiment directory state
- Standardized .txt/JSON outputs ready for downstream analysis (MetaboAnalyst, R, Python)

## How to apply

Implement the MetDataModel core classes (Spectrum, Mass track, Elution peak, Feature, Empirical compound, Study/Experiment/Sample/Method) in Python 3 with JSON-compatible structures, keeping the models minimal per the package's design goal of encouraging reuse and extension. When the pcpfm pipeline builds empirical compounds via khipu, write the grouped features with their adduct/isotope assignments as empCpd.json under the experiment's annotations directory; each downstream annotation step (e.g. MS2 mapping, then matchms-based Level 2 annotation) reads the previous JSON moniker and writes an updated one under a new moniker (e.g. 'MS2_mapped' → 'MS2_MS1_annotated'), exploiting the chainable design. Serialize the full experiment state as experiment.json alongside converted_acquisitions, asari_results, and feature_tables directories. Validation: each emitted file must parse as valid JSON and each empCpd entry must carry its constituent features plus adduct/isotopologue assignments, with singleton features retained as ungrouped entries.

## Related tools

- **MetDataModel** (Provides the minimal, JSON-compatible core data models (Spectrum, Feature, Empirical compound, Experiment) that are serialized to JSON between pipeline steps.) — https://github.com/shuzhao-li-lab/metDataModel
- **pcpfm** (Python-centric pipeline that writes and consumes the JSON monikers (empCpd.json, experiment.json) across build_empCpds, map_ms2, and l2_annotate operations.) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python 3** (Implementation language in which the MetDataModel data structures and their JSON serialization are demonstrated.)

## Examples

```
pcpfm build_empCpds --input_feature_table full_feature_table.tsv --empCpd_name empCpd  # writes annotations/empCpd.json in JSON
```

## Evaluation signals

- empCpd.json exists in the experiment's annotations directory and parses successfully as JSON.
- Each empirical compound entry contains its grouped constituent features with assigned adducts/isotopologues; singleton features without inferred adducts are retained as ungrouped entries.
- Chained annotation monikers (e.g. 'MS2_MS1_annotated') parse as JSON and each Level 2 annotation corresponds to a reference MS2 match meeting --ms2_min_peaks and the configured similarity metric.
- Experiment directory contains experiment.json alongside the expected layout (annotations/, asari_results/, converted_acquisitions/, feature_tables/, results/, QAQC_figs/).
- Serialized structures conform to the MetDataModel core concepts: peak is sample-specific, feature is experiment-specific, empCpd models a tentative metabolite possibly mixing unresolved isomers.

## Limitations

- Internal structures of each MetDataModel class are not meant to be final; long-term schema stability is not guaranteed.
- The empCpd conflates unresolved isomers: if the experiment does not resolve isomers, the empirical compound can be any or a mixture of the isomers, so JSON records carry tentative rather than definitive metabolite identity.
- Some annotation sources (HMDB, LMSD) are not redistributed and must be downloaded separately via 'pcpfm download_extras' after agreeing to license terms (HMDB is not for commercial use), so JSON annotation outputs depend on externally obtained resources.

## Evidence

- [readme] The structure of empCpd is JSON compatible and chainable: [evidence span withheld — anchor status: external_doc]
- [intro] We demonstrate their implementation in Python 3 and JSON in MetDataModel.: "We demonstrate their implementation in Python 3 and JSON in MetDataModel."
- [readme] empirical compounds as a JSON file representing putative metabolites: "This includes feature tables that are optionally blank masked, normalized, batch corrected, annotated or otherwise curated by PCPFM and empirical compounds as a JSON file representing putative"
- [intro] The goal of MetDataModel is to encourage reuse and extension, therefore the data models are kept minimal.: "The goal of MetDataModel is to encourage reuse and extension, therefore the data models are kept minimal."
- [other] empCpd.json saved under moniker with adduct/isotope pre-annotation: [evidence span withheld — anchor status: not_found]
- [readme] output data in standardized formats (.txt, JSON): "- output data in standardized formats (.txt, JSON), ready for downstream analysis"
