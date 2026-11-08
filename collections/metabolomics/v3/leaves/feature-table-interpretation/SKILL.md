---
name: feature-table-interpretation
description: Use when starting from an assembled pcpfm experiment directory containing
  converted centroided mzML files plus sample-to-file metadata, and feature tables
  (full and preferred) must be generated via Asari before QC, annotation, or normalization
  steps.
when_to_use_negative:
- The input is already a feature table (e.g. a user_created_table under feature_tables/);
  rerunning Asari on processed tables is wrong — downstream curation steps (normalization,
  blank masking) operate on the generated tables instead.
- Input files are Thermo .raw only — conversion to mzML must happen first (e.g. via
  ThermoRawFileParser) since this skill preprocesses mzML acquisitions.
- Samples in the experiment have mixed or uncertain ionization modes — pcpfm infers
  one ionization mode that must be consistent across all samples.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pcpfm
  role: CLI wrapper that runs the Asari preprocessing step on the experiment directory
    and registers the resulting 'full' and 'preferred' feature table monikers in the
    experiment object
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Asari
  role: Raw data preprocessor that converts mzML data into feature tables with unique
    feature identifiers mapped to observed m/z and retention time values
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: Python 3
  role: Implementation language of the pcpfm pipeline and Asari preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:8d6a9c71cba891624e2e9182262d1199dac16b1e2e05fa950e4fa61461bf9839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
  tools:
  - Asari
  - Python
  - Python 3
  - pcpfm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# feature-table-interpretation

## Summary

Reconstructs the Asari preprocessing step of the pcpfm pipeline that converts centroided mzML acquisitions into two feature table artifacts — the 'full' feature table (full_feature_table.tsv under export/) and the 'preferred' feature table (preferred_Feature_table.tsv) — registering both monikers in the experiment object for downstream steps.

## When to use

Use when starting from an assembled pcpfm experiment directory containing converted centroided mzML files plus sample-to-file metadata, and feature tables (full and preferred) must be generated via Asari before QC, annotation, or normalization steps.

## When NOT to use

- The input is already a feature table (e.g. a user_created_table under feature_tables/); rerunning Asari on processed tables is wrong — downstream curation steps (normalization, blank masking) operate on the generated tables instead.
- Input files are Thermo .raw only — conversion to mzML must happen first (e.g. via ThermoRawFileParser) since this skill preprocesses mzML acquisitions.
- Samples in the experiment have mixed or uncertain ionization modes — pcpfm infers one ionization mode that must be consistent across all samples.

## Inputs

- Assembled pcpfm experiment directory with converted centroided mzML acquisitions (e.g. sample1.mzML under converted_acquisitions/)
- Experiment metadata CSV/JSON mapping sample names to file paths (sequence file with Name and Path fields)
- Optional Asari parameters via --asari_command or --extra_asari

## Outputs

- preferred_Feature_table.tsv in the asari_results directory (the 'preferred' moniker feature table)
- full_feature_table.tsv under asari_results/export/ (the 'full' moniker feature table)
- Registered 'full' and 'preferred' feature table monikers in the experiment.json object

## How to apply

Point pcpfm at the assembled experiment directory containing converted centroided mzML acquisitions and the experiment metadata mapping sample names to file paths, then run the Asari preprocessing via the CLI command 'pcpfm asari -i ./my_experiment'. The pipeline automatically infers the ionization mode (positive or negative), which must be consistent across all samples and persists through subsequent processing. Non-default peak-detection settings can be passed via --asari_command or --extra_asari options. Collect the two artifacts from the asari_results directory: preferred_Feature_table.tsv and full_feature_table.tsv (under the export subdirectory), then register the 'full' and 'preferred' table monikers in the experiment object so later pipeline steps can address the tables by name. The rationale is that Asari's trackable, scalable preprocessing converts raw mzML signals into feature tables where unique feature identifiers map to observed m/z and retention time values, ready for downstream normalization, blank masking, and annotation in pcpfm.

## Related tools

- **pcpfm** (CLI wrapper that runs the Asari preprocessing step on the experiment directory and registers the resulting 'full' and 'preferred' feature table monikers in the experiment object) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Asari** (Raw data preprocessor that converts mzML data into feature tables with unique feature identifiers mapped to observed m/z and retention time values) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **Python 3** (Implementation language of the pcpfm pipeline and Asari preprocessing)

## Examples

```
pcpfm asari -i ./my_experiment
```

## Evaluation signals

- Both TSV artifacts exist: preferred_Feature_table.tsv directly under asari_results/ and full_feature_table.tsv under the export subdirectory.
- Each feature table has columns mapping unique feature identifiers to observed m/z and retention time values.
- The ionization mode inferred by pcpfm is consistent across all samples in the experiment (positive or negative).
- The 'full' and 'preferred' monikers are resolvable by name from the experiment object in later pipeline steps.
- Feature counts are consistent with Asari quality thresholds — features failing Asari quality thresholds (e.g. low signal-to-noise ratio) are excluded from the preferred table.

## Limitations

- The inferred ionization mode must be consistent across all samples; mixed-mode experiments cannot be processed as a single Asari run.
- A known issue existed where sample names that do not match their mzML file names caused failures (fixed as of 2/28/24 per the README VERSION_LOG note).
- Sample names must match their mzML file names or the pipeline may fail on older pcpfm versions; support for .raw input is limited, so conversion to mzML is expected.
- Features excluded by Asari's quality threshold (low signal-to-noise ratio) will be missing from the preferred feature table.

## Evidence

- [results] After asari is run in the pipeline, two feature table monikers are generated: 'full' and 'preferred', for the full and preferred feature tables respectively.: [evidence span withheld — anchor status: not_found]
- [other] Execute the Asari preprocessing via the pcpfm CLI command 'pcpfm asari -i ./my_experiment', letting the pipeline automatically infer the ionization mode (positive or negative): [evidence span withheld — anchor status: not_found]
- [readme] process mzML data to feature tables (Asari): "- process mzML data to feature tables (Asari)"
- [readme] asari_results/ preferred_Feature_table.tsv export/ full_feature_table.tsv: "      asari_results/
          preferred_Feature_table.tsv
          export/
              full_feature_table.tsv"
- [intro] the preprocessing tool that, among other functions, converts raw data into feature tables representing signals of interest likely to represent metabolites: "the preprocessing tool that, among other functions, converts raw data into feature tables representing signals of interest likely to represent metabolites"
- [results] The sole missing feature is due to low signal-to-noise ratio, not passing Asari quality threshold (Fig 5B).: "The sole missing feature is due to low signal-to-noise ratio, not passing Asari quality threshold (Fig 5B)."
