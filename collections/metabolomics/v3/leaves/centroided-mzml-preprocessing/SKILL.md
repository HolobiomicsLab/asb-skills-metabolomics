---
name: centroided-mzml-preprocessing
description: Use when starting from converted centroided mzML files plus experiment
  metadata (sample names mapped to file paths) and raw LC-MS data must be turned into
  m/z/RT-annotated feature tables before QC or annotation.
when_to_use_negative:
- The input is already a feature table or preprocessed peak list rather than raw/centroided
  mzML acquisitions.
- The experiment mixes samples with inconsistent ionization modes — pcpfm infers a
  single mode that must be consistent across all samples.
- Input files are Thermo .raw without conversion; pcpfm's .raw support is limited,
  so convert to mzML first (e.g. via ThermoRawFileParser).
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: pcpfm
  role: CLI wrapper that invokes Asari on the experiment directory, infers ionization
    mode, and registers the resulting 'full' and 'preferred' feature tables in the
    experiment object
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: raw data preprocessor that converts centroided mzML into feature tables; supports
    non-default peak-detection settings via --asari_command/--extra_asari
  repo: https://github.com/shuzhao-li-lab/asari_pcpfm_tutorials
- name: Python 3
  role: implementation language of the pcpfm pipeline and Asari preprocessing tool
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:f6255f842935a6da452a2ee28230a802ba2595b76cfb2b4fc7e9e2a276b49888
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - Python 3
  - pcpfm
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5666666666666667
n_claims: 30
---

# centroided-mzml-preprocessing

## Summary

Preprocess centroided mzML acquisitions into feature tables using Asari via the pcpfm CLI, producing both a 'full' and a 'preferred' feature table registered as named monikers in the experiment object for downstream pcpfm steps.

## When to use

Use when starting from converted centroided mzML files plus experiment metadata (sample names mapped to file paths) and raw LC-MS data must be turned into m/z/RT-annotated feature tables before QC or annotation.

## When NOT to use

- The input is already a feature table or preprocessed peak list rather than raw/centroided mzML acquisitions.
- The experiment mixes samples with inconsistent ionization modes — pcpfm infers a single mode that must be consistent across all samples.
- Input files are Thermo .raw without conversion; pcpfm's .raw support is limited, so convert to mzML first (e.g. via ThermoRawFileParser).

## Inputs

- centroided mzML acquisitions (converted_acquisitions directory, e.g. sample1.mzML, sample2.mzML)
- experiment metadata CSV mapping sample names to file paths (sequence/sample sheet with sample type, e.g. BLANK/QC/STD/Unknown)
- experiment directory with experiment.json

## Outputs

- preferred_Feature_table.tsv in asari_results/
- full_feature_table.tsv in asari_results/export/
- 'full' and 'preferred' feature table monikers registered in the experiment object (experiment.json)

## How to apply

Point pcpfm at the assembled experiment directory that holds the converted centroided mzML acquisitions and experiment metadata (a CSV mapping sample names to file paths), then run 'pcpfm asari -i ./my_experiment'. The pipeline automatically infers the ionization mode (positive or negative), which must be consistent across all samples and persists through subsequent processing; non-default peak-detection settings can be passed via --asari_command or --extra_asari. Collect the two artifacts from asari_results: preferred_Feature_table.tsv and the full feature table (full_feature_table.tsv under the export subdirectory). Register the 'full' and 'preferred' table monikers in the experiment object so later pcpfm steps (pre-annotation, annotation, QC, normalization) can address these tables by name. Validate by confirming both TSVs exist with columns mapping unique feature identifiers to observed m/z and retention time values.

## Related tools

- **pcpfm** (CLI wrapper that invokes Asari on the experiment directory, infers ionization mode, and registers the resulting 'full' and 'preferred' feature tables in the experiment object) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (raw data preprocessor that converts centroided mzML into feature tables; supports non-default peak-detection settings via --asari_command/--extra_asari) — https://github.com/shuzhao-li-lab/asari_pcpfm_tutorials
- **Python 3** (implementation language of the pcpfm pipeline and Asari preprocessing tool)

## Examples

```
pcpfm asari -i ./my_experiment
```

## Evaluation signals

- Both preferred_Feature_table.tsv and asari_results/export/full_feature_table.tsv exist in the experiment directory.
- Each feature table has columns mapping unique feature identifiers to observed m/z and retention time values.
- The ionization mode inferred by pcpfm is consistent across all samples in the experiment.
- The 'full' and 'preferred' monikers resolve by name when referenced in later pcpfm pipeline steps.
- Asari quality threshold behavior is expected: low signal-to-noise features are excluded from the preferred table (the sole missing feature in the Bowen et al 2023 comparison was due to not passing the Asari quality threshold).

## Limitations

- Sample names that do not match their mzML file names caused an issue, fixed as of 2/28/24 per the README VERSION_LOG.
- The inferred ionization mode must be consistent across all samples and persists throughout subsequent processing.
- Support for Thermo .raw input is limited; .raw support beyond conversion to mzML is minimal.

## Evidence

- [readme] process mzML data to feature tables (Asari): "process mzML data to feature tables (Asari)"
- [readme] asari_results preferred and full feature tables: "asari_results/
          preferred_Feature_table.tsv
          export/
              full_feature_table.tsv"
- [readme] inputs are raw files and metadata csv: "Inputs should include a set of raw files (.raw or .mzML) and a csv file for metadata (minimal sample names and file path)."
- [results] Asari quality threshold excludes low SNR feature: "The sole missing feature is due to low signal-to-noise ratio, not passing Asari quality threshold (Fig 5B)."
- [intro] preprocessing tool converts raw data into feature tables: "the preprocessing tool that, among other functions, converts raw data into feature tables representing signals of interest likely to represent metabolites"
