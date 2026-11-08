---
name: elution-peak-detection
description: Use when raw centroided LC-MS mzML acquisitions must be preprocessed
  into feature tables with per-sample elution peaks and m/z–retention-time annotated
  features, e.g. inside a pcpfm experiment directory.
when_to_use_negative:
- The input is already a feature table or empirical-compound JSON — peak detection
  has already been performed and downstream QC/annotation steps apply instead.
- The dataset mixes positive- and negative-ionization-mode samples; pcpfm requires
  an ionization mode consistent across all samples.
- Data types outside LC-MS scope, e.g. GC-MS, for which pcpfm/Asari support is explicitly
  still being added.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Asari
  role: raw-data preprocessor that builds mass tracks, detects elution peaks, and
    exports the full and preferred feature tables
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: pcpfm
  role: pipeline CLI that wraps Asari, infers ionization mode, writes preferred/full
    tables into asari_results, and registers table monikers in the experiment
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: metDataModel
  role: defines the Mass track, Elution peak, and Feature data structures underlying
    the peak-to-feature aggregation
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:f0e119d67464afc7717250687ddc084cea998e00375cdbc1f299c9c5fac77943
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - Python 3
  - metDataModel
  - pcpfm
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# elution-peak-detection

## Summary

Detection of elution peaks on mass tracks (full-retention-time extracted ion chromatograms) by Asari, converting centroided mzML acquisitions into sample-level peaks that are aggregated into experiment-level features and exported as pcpfm 'full' and 'preferred' feature tables.

## When to use

Use when raw centroided LC-MS mzML acquisitions must be preprocessed into feature tables with per-sample elution peaks and m/z–retention-time annotated features, e.g. inside a pcpfm experiment directory.

## When NOT to use

- The input is already a feature table or empirical-compound JSON — peak detection has already been performed and downstream QC/annotation steps apply instead.
- The dataset mixes positive- and negative-ionization-mode samples; pcpfm requires an ionization mode consistent across all samples.
- Data types outside LC-MS scope, e.g. GC-MS, for which pcpfm/Asari support is explicitly still being added.

## Inputs

- centroided mzML acquisition files (e.g. sample1.mzML, sample2.mzML in converted_acquisitions/)
- experiment directory with metadata CSV mapping sample names to file paths (Sample Type, Name, Filepath fields)
- optional non-default Asari peak-detection parameters via --asari_command / --extra_asari

## Outputs

- asari_results/preferred_Feature_table.tsv (preferred feature table)
- asari_results/export/full_feature_table.tsv (full feature table)
- feature table columns mapping unique feature identifiers to observed m/z and retention time values
- 'full' and 'preferred' table monikers registered in the experiment object (experiment.json)

## How to apply

Point pcpfm at an assembled experiment directory containing converted centroided mzML files and metadata mapping sample names to file paths, then run Asari via `pcpfm asari -i ./my_experiment`; pcpfm infers the ionization mode (positive or negative), which must be consistent across all samples and persists through later steps. Asari first builds mass tracks — extracted ion chromatograms spanning the full retention time — then detects elution peaks on each track; each peak is specific to a sample, while a feature is specific to an experiment, so per-sample peaks are aggregated into features with observed m/z and retention time. Non-default peak-detection settings can be passed via --asari_command or --extra_asari. Collect the two artifacts: preferred_Feature_table.tsv directly under asari_results and full_feature_table.tsv under the export subdirectory, and register the 'full' and 'preferred' monikers in the experiment object so later pipeline steps can address the tables by name.

## Related tools

- **Asari** (raw-data preprocessor that builds mass tracks, detects elution peaks, and exports the full and preferred feature tables) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **pcpfm** (pipeline CLI that wraps Asari, infers ionization mode, writes preferred/full tables into asari_results, and registers table monikers in the experiment) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **metDataModel** (defines the Mass track, Elution peak, and Feature data structures underlying the peak-to-feature aggregation) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm asari -i ./my_experiment
```

## Evaluation signals

- Both asari_results/preferred_Feature_table.tsv and asari_results/export/full_feature_table.tsv exist after the run.
- Feature table columns map unique feature identifiers to observed m/z and retention time values.
- The ionization mode inferred by pcpfm is consistent across all samples in the experiment.
- Features missing from the expected output are attributable to low signal-to-noise ratio failing the Asari quality threshold, not pipeline failure.
- The 'full' and 'preferred' monikers are registered in the experiment object and resolvable by name in later pcpfm steps.

## Limitations

- Low signal-to-noise features do not pass the Asari quality threshold and are dropped, so a genuinely present metabolite can be missing from the feature table.
- The inferred ionization mode must be consistent across all samples and persists through subsequent processing.
- Sample names that do not match their mzML file names caused an issue, fixed as of 2/28/24 per the VERSION_LOG.

## Evidence

- [readme] Mass track is a concept started in Asari: [evidence span withheld — anchor status: external_doc]
- [readme] peak vs feature: [evidence span withheld — anchor status: external_doc]
- [intro] raw data to feature tables: "the preprocessing tool that, among other functions, converts raw data into feature tables representing signals of interest likely to represent metabolites"
- [results] Asari quality threshold: "The sole missing feature is due to low signal-to-noise ratio, not passing Asari quality threshold (Fig 5B)."
- [readme] process mzML data to feature tables: "process mzML data to feature tables (Asari)"
