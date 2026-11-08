---
name: lc-ms-feature-detection
description: Use when starting from centroided mzML (or Thermo .raw) acquisitions
  and an experiment directory with sample-name-to-file-path metadata, and you need
  feature tables (m/z + retention time per feature) for downstream QC, normalization,
  or annotation.
when_to_use_negative:
- Input is already a feature table or otherwise preprocessed output — this skill operates
  on raw mzML/.raw acquisitions
- The study mixes ionization modes across samples, since pcpfm infers a single ionization
  mode that must be consistent across all samples
- GC-MS or other non-LC-MS data types, which pcpfm does not yet support (support is
  under development)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Asari
  role: Raw-data preprocessor that converts mzML acquisitions into feature tables
    via mass-track based peak detection
  repo: https://github.com/shuzhao-li/asari
- name: pcpfm
  role: Python-centric pipeline CLI that invokes Asari on the experiment directory,
    registers the 'full' and 'preferred' feature-table monikers, and organizes outputs
    under asari_results
  repo: https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- name: metDataModel
  role: Defines the core data structures (mass track, elution peak, feature, empirical
    compound) that underlie the feature detection output
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011912
    title: Common data models to streamline metabolomics processing and annotation,
      and implementation in a Python pipeline
  source_content_hash: sha256:f2b36e77093c5b4cba17cba8a49be78e5c4741feedf2938ee5a1a736768ae3d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Asari
  - Jupyter
  - OpenMS
  - Python
  - Python 3
  - UmetaFlow
  - metDataModel
  - pcpfm
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.68
n_claims: 25
---

# lc-ms-feature-detection

## Summary

Detect LC-MS features from raw mzML acquisitions using Asari, the Python preprocessing tool in the pcpfm pipeline, producing 'full' and 'preferred' feature tables that map unique feature identifiers to observed m/z and retention time values.

## When to use

Use when starting from centroided mzML (or Thermo .raw) acquisitions and an experiment directory with sample-name-to-file-path metadata, and you need feature tables (m/z + retention time per feature) for downstream QC, normalization, or annotation.

## When NOT to use

- Input is already a feature table or otherwise preprocessed output — this skill operates on raw mzML/.raw acquisitions
- The study mixes ionization modes across samples, since pcpfm infers a single ionization mode that must be consistent across all samples
- GC-MS or other non-LC-MS data types, which pcpfm does not yet support (support is under development)

## Inputs

- Centroided mzML files (converted from Thermo .raw via ThermoRawFileParser) located in the experiment directory's converted_acquisitions folder
- Experiment directory with experiment.json metadata mapping sample names to file paths
- Optional Asari parameter overrides (--asari_command / --extra_asari)

## Outputs

- preferred_Feature_table.tsv in the asari_results directory (registered under the 'preferred' moniker)
- full_feature_table.tsv under asari_results/export (registered under the 'full' moniker)
- Feature table TSVs with columns mapping unique feature identifiers to observed m/z and retention time values
- An experiment object with the 'full' and 'preferred' feature-table monikers registered

## How to apply

Point pcpfm at the assembled experiment directory containing converted centroided mzML files and the experiment metadata mapping sample names to file paths, then run Asari preprocessing via the CLI (e.g. 'pcpfm asari -i ./my_experiment'). The pipeline automatically infers the ionization mode (positive or negative), which must be consistent across all samples and persists through subsequent processing; pass non-default peak-detection settings via --asari_command or --extra_asari options if needed. Collect the two generated artifacts from the asari_results directory: the preferred feature table (preferred_Feature_table.tsv) and the full feature table (full_feature_table.tsv under the export subdirectory), and register the 'full' and 'preferred' monikers in the experiment object so later pipeline steps can address the tables by name. Validation confirms both TSVs exist with columns mapping unique feature identifiers to observed m/z and retention time values, and that the inferred ionization mode is consistent across all samples. Asari organizes signals as mass tracks (extracted ion chromatograms spanning the full retention time) and elutes peaks per sample, while a feature is specific to an experiment, which is why the pipeline yields a full table and a preferred table.

## Related tools

- **Asari** (Raw-data preprocessor that converts mzML acquisitions into feature tables via mass-track based peak detection) — https://github.com/shuzhao-li/asari
- **pcpfm** (Python-centric pipeline CLI that invokes Asari on the experiment directory, registers the 'full' and 'preferred' feature-table monikers, and organizes outputs under asari_results) — https://github.com/shuzhao-li-lab/PythonCentricPipelineForMetabolomics
- **metDataModel** (Defines the core data structures (mass track, elution peak, feature, empirical compound) that underlie the feature detection output) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
pcpfm asari -i ./my_experiment
```

## Evaluation signals

- Both preferred_Feature_table.tsv and full_feature_table.tsv exist in the expected asari_results directory layout (full table under the export subdirectory)
- Feature table TSVs contain columns mapping unique feature identifiers to observed m/z and retention time values
- The ionization mode inferred by pcpfm is consistent across all samples in the experiment
- The 'full' and 'preferred' monikers are registered in the experiment object so the tables can be addressed by name in later pipeline steps
- Downstream steps (e.g. outlier removal by |Z| > 2.5 on number-of-features) can consume the tables without schema errors

## Limitations

- A single ionization mode (positive or negative) is inferred and must be consistent across all samples; mixed-mode experiments are not handled
- Sample names that do not match their mzML file names were a known bug, fixed as of 2/28/24 per the README's version log note
- Support for GC and other data types is not yet available; .raw files can be used but support is limited

## Evidence

- [other] After asari is run in the pipeline, two feature table monikers are generated: 'full' and 'preferred', for the full and preferred feature tables respectively.: [evidence span withheld — anchor status: not_found]
- [abstract] The pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation.: "Our pipeline takes experimental data in .mzML or .raw format and outputs annotated feature tables for subsequent biological interpretation."
- [readme] Mass track is a concept started in Asari (https://www.nature.com/articles/s41467-023-39889-1), as extracted ion chromatogram spanning the full retention time.: [evidence span withheld — anchor status: external_doc]
- [readme] The pipeline can - convert Thermo .raw to mzML (ThermoRawFileParser) - process mzML data to feature tables (Asari): "convert Thermo .raw to mzML (ThermoRawFileParser)
- process mzML data to feature tables (Asari)"
- [readme] A peak is specific to a sample, while a feature is specific to an experiment.: [evidence span withheld — anchor status: external_doc]
