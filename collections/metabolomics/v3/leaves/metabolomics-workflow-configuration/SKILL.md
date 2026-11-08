---
name: metabolomics-workflow-configuration
description: Use when setting up an untargeted LC-MS/MS processing run (raw or mzML
  inputs, blanks/QC present, positive or negative ionization) and you must decide
  which UmetaFlow steps to enable and how to list samples.
when_to_use_negative:
- Your data is already a processed feature table rather than raw LC-MS files — pre-processing/re-quantification
  toggles and file conversion are irrelevant.
- You only need to run the Jupyter/pyOpenMS variant interactively without Snakemake
  orchestration (use the pyOpenMS_UmetaFlow repo instead).
- You want to change algorithm behavior of OpenMS tools themselves rather than enable/disable
  or parameterize existing workflow rules.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: UmetaFlow
  role: The Snakemake-implemented untargeted metabolomics workflow being configured
    (config.yaml toggles and parameter blocks).
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: Snakemake
  role: Workflow engine whose config schema the config.yaml file controls; dry-run
    validates the configuration.
- name: OpenMS 3.0
  role: Toolset invoked by the configured pre-processing rules (feature detection,
    map alignment, adduct annotation, re-quantification).
- name: ProteoWizard
  role: External converter to use instead of the workflow's file-conversion rule for
    Agilent/Bruker vendor files.
- name: Python (data_files.py)
  role: Generates config/dataset.tsv from files in data/raw/ or data/mzML/.
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:37bf80c9fcc591bd6612b3b5636e4ef3944da3a370642a8b62236d0e5f051eaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - Jupyter
  - OpenMS
  - OpenMS 3.0
  - ProteoWizard
  - Python
  - Python (data_files.py)
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-workflow-configuration

## Summary

Configure the UmetaFlow untargeted metabolomics workflow by editing config/config.yaml (TRUE/FALSE toggles for file conversion, pre-processing, re-quantification, SIRIUS/GNPS/spectral matching), generating dataset.tsv/blanks.tsv/samples.tsv, and running a Snakemake dry-run before execution.

## When to use

Use when setting up an untargeted LC-MS/MS processing run (raw or mzML inputs, blanks/QC present, positive or negative ionization) and you must decide which UmetaFlow steps to enable and how to list samples.

## When NOT to use

- Your data is already a processed feature table rather than raw LC-MS files — pre-processing/re-quantification toggles and file conversion are irrelevant.
- You only need to run the Jupyter/pyOpenMS variant interactively without Snakemake orchestration (use the pyOpenMS_UmetaFlow repo instead).
- You want to change algorithm behavior of OpenMS tools themselves rather than enable/disable or parameterize existing workflow rules.

## Inputs

- Thermo .raw files placed in data/raw/ (or vendor files pre-converted to centroided mzML placed in data/mzML/)
- config/config.yaml (rule toggles and ionisation-mode/parameter settings)
- Sample name lists: config/dataset.tsv, config/blanks.tsv, config/samples.tsv

## Outputs

- A validated Snakemake configuration (dry-run resolving all rules) ready for 'snakemake --use-conda --cores all'
- Centroided mzML files under data/mzML/ (when file conversion is enabled)
- TSV results produced under results/ per the enabled workflow steps

## How to apply

Clone snakemake_UmetaFlow and create the 'umetaflow-snakemake' conda environment, then edit config/config.yaml writing TRUE/FALSE per rule: set file conversion to TRUE for Thermo files (ThermoRawFileParser converts data/raw/ to centroided mzML), but for Agilent/Bruker data write FALSE for that rule and place ProteoWizard-converted mzML files in data/mzML/. Set the dataset-specific ionisation mode and other commented parameters (e.g. positive/negative) and optionally enable blank/QC removal and the 'minfrac' filter that removes consensus features with too many missing values, plus re-quantification of features with missing values. Populate data/raw/ (or data/mzML/) and generate config/dataset.tsv via 'python data_files.py' or the Create_dataset_tsv notebook, splitting blanks and true samples into config/blanks.tsv and config/samples.tsv. Validate with 'snakemake --use-conda --dry-run' before executing 'snakemake --use-conda --cores all' inside the snakemake_UmetaFlow directory.

## Related tools

- **UmetaFlow** (The Snakemake-implemented untargeted metabolomics workflow being configured (config.yaml toggles and parameter blocks).) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **Snakemake** (Workflow engine whose config schema the config.yaml file controls; dry-run validates the configuration.)
- **OpenMS 3.0** (Toolset invoked by the configured pre-processing rules (feature detection, map alignment, adduct annotation, re-quantification).)
- **ProteoWizard** (External converter to use instead of the workflow's file-conversion rule for Agilent/Bruker vendor files.)
- **Python (data_files.py)** (Generates config/dataset.tsv from files in data/raw/ or data/mzML/.) — https://github.com/biosustain/snakemake_UmetaFlow.git

## Examples

```
git clone https://github.com/biosustain/snakemake_UmetaFlow.git && python data_files.py && snakemake --use-conda --dry-run
```

## Evaluation signals

- 'snakemake --use-conda --dry-run' completes without errors, confirming the config, schemas and dataset.tsv are consistent.
- config/config.yaml has every rule either TRUE or FALSE and ionisation mode matching the instrument method (positive/negative) — no ambiguous entries.
- config/dataset.tsv rows match actual files under data/raw/ or data/mzML/; blanks.tsv and samples.tsv partition the sample list exactly.
- Thermo vendor files are absent from data/raw/ when file conversion is FALSE, and converted centroided mzML files appear when it is TRUE.
- Optional filters (blank/QC removal, minfrac) are only enabled when the corresponding blanks/QC entries exist in blanks.tsv.

## Limitations

- macOS Silicon installs need a manual macos_setup.py, manual OpenMS 3.3.0 download with codesigning, and PATH export for TOPP tools; the script modifies workflow code and conda files and should not be committed.
- Only Thermo files are auto-converted; Agilent/Bruker/other vendor files must be converted independently via ProteoWizard before being placed in data/mzML/.
- Wrong TRUE/FALSE toggles or missing blank definitions silently change which rules run or which features get filtered, so dry-run validation is required before full execution.

## Evidence

- [readme] Add all your files in the `data/raw/` or `data/mzML/` directory and generate the `dataset.tsv` table: "Add all your files in the `data/raw/` or `data/mzML/` directory and generate the `dataset.tsv` table to specify the samples (filenames) that will be processed."
- [readme] Write <span style="color: green">TRUE</span>/<span style="color: red">FALSE</span> if you want to run/skip a specific rules: "The `config.yaml` file determines the workflow steps: - Write <span style="color: green">TRUE</span>/<span style="color: red">FALSE</span> if you want to run/skip a specific rules of the workflow."
- [readme] Test your configuration by performing a dry-run via: "Test your configuration by performing a dry-run via

    snakemake --use-conda --dry-run"
- [readme] This step includes an optional removal of blank/QC samples: "This step includes an optional removal of blank/QC samples if defined by the user. Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [discussion] generate config/dataset.tsv via the provided 'python data_files.py' utility: [evidence span withheld — anchor status: not_found]
- [abstract] combines algorithms for data pre-processing: "combines algorithms for data pre-processing, spectral matching, molecular formula and structural predictions"
- [intro] using OpenMS [17] 3.0 tools for feature detection: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
- [abstract] UmetaFlow is implemented as a Snakemake workflow: "UmetaFlow is implemented as a Snakemake workflow, making it easy to use, scalable, and reproducible."
