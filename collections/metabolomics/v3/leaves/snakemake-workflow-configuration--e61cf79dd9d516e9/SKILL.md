---
name: snakemake-workflow-configuration
description: 'Use when starting an untargeted LC-MS/MS analysis with UmetaFlow: you
  have Thermo raw files or mzMLs, need to set TRUE/FALSE step flags, ionisation mode,
  minfrac thresholds and sample/blank tables in the config/ folder.'
when_to_use_negative:
- Input data are already processed feature tables (e.g. a pre-computed feature quantification
  matrix) rather than raw or mzML files — configuration of the detection/conversion
  pipeline is unnecessary.
- The user needs a single ad-hoc algorithm run (e.g. only FeatureFinderMetaboIdent)
  rather than the full chained workflow; the pyOpenMS notebooks alone may suffice.
- 'Running on Windows: the Snakemake implementation targets Linux and MacOS systems,
  so use the pyOpenMS Jupyter implementation instead.'
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: UmetaFlow
  role: The Snakemake workflow being configured; config.yaml controls which of its
    seven-plus steps run
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: Snakemake
  role: Workflow engine executing the configured DAG (--use-conda --cores all; ≥7.14.0)
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: pyOpenMS
  role: Python bindings powering the equivalent Jupyter-notebook implementation of
    the same configured pipeline
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow.git
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:5810f4767c5130784a9a66fcfa96f6edcfa6a3e2f3f708efe29a75a198b742cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - Jupyter
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# snakemake-workflow-configuration

## Summary

Configure the UmetaFlow Snakemake workflow (or its pyOpenMS Jupyter equivalent) for a specific untargeted metabolomics dataset by editing config.yaml step flags and parameters and generating dataset.tsv, blanks.tsv and samples.tsv, so that feature detection, re-quantification, annotation and benchmarking steps run reproducibly.

## When to use

Use when starting an untargeted LC-MS/MS analysis with UmetaFlow: you have Thermo raw files or mzMLs, need to set TRUE/FALSE step flags, ionisation mode, minfrac thresholds and sample/blank tables in the config/ folder.

## When NOT to use

- Input data are already processed feature tables (e.g. a pre-computed feature quantification matrix) rather than raw or mzML files — configuration of the detection/conversion pipeline is unnecessary.
- The user needs a single ad-hoc algorithm run (e.g. only FeatureFinderMetaboIdent) rather than the full chained workflow; the pyOpenMS notebooks alone may suffice.
- Running on Windows: the Snakemake implementation targets Linux and MacOS systems, so use the pyOpenMS Jupyter implementation instead.

## Inputs

- Thermo .raw files placed in data/raw/ (converted to centroided mzML) or vendor mzML files from Agilent/Bruker placed in data/mzML/
- config/config.yaml with TRUE/FALSE step flags and dataset parameters (e.g. positive/negative ionisation, minfrac)
- config/dataset.tsv listing sample_name and comment per MS file
- config/blanks.tsv and config/samples.tsv defining blank/QC and biological samples (optional)

## Outputs

- A validated, executable Snakemake DAG (results in results/ as .TSV files, interim files under results/interim/)
- Pre-processed and optionally re-quantified feature tables (consensusXML/TSV) for downstream GNPS, SIRIUS/CSI:FingerID and benchmarking steps

## How to apply

Clone the workflow (git clone https://github.com/biosustain/snakemake_UmetaFlow.git), install conda/mamba and create the umetaflow-snakemake environment (conda create -c conda-forge -c bioconda -n umetaflow-snakemake snakemake python=3.12), then edit config/config.yaml: write TRUE/FALSE to run or skip each rule (e.g. file conversion, re-quantification, SIRIUS, GNPS, spectral matching, MS2Query) and set dataset-specific parameters such as positive/negative ionisation and the optional minfrac threshold for dropping consensus features with too many missing values. Place raw data in data/raw/ (Thermo files converted to centroided mzML via ThermoRawFileParser) or pre-converted mzMLs in data/mzML/ (for Agilent/Bruker, convert externally with proteowizard), and generate config/dataset.tsv plus, if blanks/QCs exist, config/blanks.tsv and config/samples.tsv using the Create_dataset_tsv notebook or python data_files.py. Validate with a dry run (snakemake --use-conda --dry-run) before executing the full run; on MacOS Silicon, run python macos_setup.py and install OpenMS 3.3.0 manually, re-signing the binaries with codesign. The same configuration logic (step flags, minfrac, sample tables) maps onto the pyOpenMS Jupyter notebook implementation.

## Related tools

- **UmetaFlow** (The Snakemake workflow being configured; config.yaml controls which of its seven-plus steps run) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **Snakemake** (Workflow engine executing the configured DAG (--use-conda --cores all; ≥7.14.0)) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **pyOpenMS** (Python bindings powering the equivalent Jupyter-notebook implementation of the same configured pipeline) — https://github.com/biosustain/pyOpenMS_UmetaFlow.git

## Examples

```
snakemake --use-conda --dry-run && snakemake --use-conda --cores all
```

## Evaluation signals

- A dry run (snakemake --use-conda --dry-run) resolves the full DAG without missing-input or syntax errors.
- config/dataset.tsv, blanks.tsv and samples.tsv are mutually consistent: every sample_name in blanks/samples appears in dataset.tsv.
- Steps set to FALSE in config.yaml (e.g. file conversion for Agilent/Bruker mzMLs supplied externally) are skipped, and all data files exist under data/raw/ or data/mzML/ as the flags imply.
- The executed run produces .TSV results under results/ and interim consensusXML files under results/interim/ for the enabled steps.
- On benchmark datasets like MTBLS733, the configured pipeline reproduces the expected behaviour (e.g. detection of 778 of 836 targeted features, 93.1% identification rate).

## Limitations

- The Snakemake implementation runs on Linux and MacOS only; MacOS Silicon requires the macos_setup.py script, a manual OpenMS 3.3.0 install, and manual re-signing of binaries (sudo codesign --force -s - ./bin/*).
- Non-Thermo vendor files must be converted externally with proteowizard before configuration; the built-in ThermoRawFileParser conversion rule only covers Thermo raw files.
- The article does not state software version numbers for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID or Snakemake in the main text, so exact reproducibility of benchmark numbers may depend on versions from the repository environment files.

## Evidence

- [readme] The config.yaml file determines the workflow steps: Write TRUE/FALSE if you want to run/skip a specific rules of the workflow.: [evidence span withheld — anchor status: not_found]
- [readme] Optional minfrac step allows removal of consensus features with too many missing values in both pre-processing and re-quantification.: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [methods] Configure the UmetaFlow Snakemake workflow by setting TRUE/FALSE step flags and parameters (e.g. positive/negative ionisation) in config.yaml and generating dataset.tsv, blanks.tsv and samples.tsv.: [evidence span withheld — anchor status: not_found]
- [abstract] UmetaFlow is implemented as a Snakemake workflow, making it easy to use, scalable, and reproducible.: "UmetaFlow is implemented as a Snakemake workflow, making it easy to use, scalable, and reproducible."
- [readme] For non-Thermo vendor data, conversion must be done externally with proteowizard and files placed in data/mzML/.: "If you have Agilent, Bruker, or other vendor files, skip that step (write "FALSE" for rule file conversion in the config.yaml file - see more under "Configure workflow")"
