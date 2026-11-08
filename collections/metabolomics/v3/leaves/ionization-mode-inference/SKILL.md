---
name: ionization-mode-inference
description: Use when starting pcpfm processing on an assembled experiment directory
  of mzML acquisitions and the ionization mode is unknown or unrecorded, and it must
  be set once and applied uniformly across all samples.
when_to_use_negative:
- The experiment mixes positive- and negative-mode samples — the inferred mode must
  be consistent across all samples, so mixed-mode acquisitions should be split into
  separate experiments first
- Input is already a feature table or annotated empCpd JSON — this skill operates
  during Asari preprocessing of raw/centroided mzML data
- Data is GC-MS or another non-LC-MS type — the PCPFM README notes support for GC
  and other data types is still in development
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pcpfm
  role: CLI wrapper that runs the Asari preprocessing step and auto-infers the experiment-wide
    ionization mode
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Raw-data preprocessor that converts mzML data into feature tables under the
    inferred ionization mode
- name: Python 3
  role: Implementation language of the pcpfm pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:73011d5686627b75586add98adf2fde5410a26094b46c7f09e01306016e39d51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - Python 3
  - pcpfm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ionization-mode-inference

## Summary

Automatically infers the ionization mode (positive or negative) of an LC-MS experiment via the pcpfm/Asari preprocessing step on centroided mzML files, so the mode is fixed consistently across all samples and persists through feature-table generation and downstream annotation.

## When to use

Use when starting pcpfm processing on an assembled experiment directory of mzML acquisitions and the ionization mode is unknown or unrecorded, and it must be set once and applied uniformly across all samples.

## When NOT to use

- The experiment mixes positive- and negative-mode samples — the inferred mode must be consistent across all samples, so mixed-mode acquisitions should be split into separate experiments first
- Input is already a feature table or annotated empCpd JSON — this skill operates during Asari preprocessing of raw/centroided mzML data
- Data is GC-MS or another non-LC-MS type — the PCPFM README notes support for GC and other data types is still in development

## Inputs

- Assembled experiment directory with converted, centroided mzML acquisitions
- Experiment metadata mapping sample names to file paths (CSV/sequence file)
- Optional Asari parameters (--asari_command / --extra_asari)

## Outputs

- preferred_Feature_table.tsv (preferred feature table) in asari_results/
- full_feature_table.tsv under the export subdirectory (full feature table)
- Registered 'full' and 'preferred' table monikers in the experiment object (experiment.json)
- Fixed ionization mode (positive or negative) applied to all downstream processing

## How to apply

Point pcpfm at the assembled experiment directory containing converted centroided mzML acquisitions and experiment metadata mapping sample names to file paths, then run the 'pcpfm asari' step, letting the pipeline automatically infer the ionization mode (positive or negative) from the data. The inferred mode must be consistent across all samples in the experiment — verify this rather than overriding per-sample; if samples differ in mode they should be split into separate experiments. Non-default peak-detection settings can be passed via --asari_command or --extra_asari options. The mode inferred here persists throughout subsequent processing: the step produces 'preferred' (preferred_Feature_table.tsv) and 'full' (full_feature_table.tsv under the export subdirectory) feature table monikers that are registered in the experiment object so later pipeline steps (pre-annotation via khipu, MS1/MS2 annotation) can address them by name.

## Related tools

- **pcpfm** (CLI wrapper that runs the Asari preprocessing step and auto-infers the experiment-wide ionization mode) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Raw-data preprocessor that converts mzML data into feature tables under the inferred ionization mode)
- **Python 3** (Implementation language of the pcpfm pipeline)

## Examples

```
pcpfm asari -i ./my_experiment
```

## Evaluation signals

- Both preferred_Feature_table.tsv and the full_feature_table.tsv under the export subdirectory exist in asari_results/
- Feature tables have columns mapping unique feature identifiers to observed m/z and retention time values
- The ionization mode inferred by pcpfm is consistent across all samples in the experiment
- The 'full' and 'preferred' monikers are registered in the experiment object and resolvable by name in later pipeline steps

## Limitations

- The inferred ionization mode must be consistent across all samples — a single experiment cannot contain mixed modes
- Support for .raw files is limited; conversion to mzML (e.g. via ThermoRawFileParser) is the expected input path
- Sample names that do not match their mzML file names were a known issue fixed as of 2/28/24 per the README VERSION_LOG note

## Evidence

- [other] ionization mode inference via pcpfm asari: [evidence span withheld — anchor status: not_found]
- [other] mode persists through processing: [evidence span withheld — anchor status: not_found]
- [readme] asari produces feature tables: "process mzML data to feature tables (Asari)"
- [abstract] pipeline input formats: "Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation."
