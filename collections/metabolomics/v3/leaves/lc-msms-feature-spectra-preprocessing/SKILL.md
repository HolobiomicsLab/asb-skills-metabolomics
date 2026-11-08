---
name: lc-msms-feature-spectra-preprocessing
description: Use when an LC-MS/MS untargeted metabolomics dataset (e.g. a Kathib et
  al. 2024 feature table with quantification, metadata, and MS/MS spectra) must be
  prepared for msFeaST spectral similarity computation.
when_to_use_negative:
- The goal is only to inspect pre-processed example files in the interactive dashboard
  — the README states only the msFeaST_Dashboard_bundle.html and a ready-made dashboard_data.json
  are needed, with no preprocessing.
- Input is already the msFeaST similarity/distance matrix or dashboard_data.json;
  this skill concerns the upstream preprocessing of quantification, metadata, and
  spectral data.
- Data are not LC-MS/MS untargeted metabolomics feature/spectral tables and cannot
  supply the quantification table, metadata table, and MS/MS spectra the pipeline
  requires.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: msFeaST
  role: Python/R/browser workflow whose preprocessing notebook processes the quantification
    table, metadata table, and spectral data into processed MS/MS spectra for spectral
    similarity scoring.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:0bca8f36ab2fa5bd1d2fbab93ea2ca372671f1ba8adae49c979c4d1c2103108f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lc-msms-feature-spectra-preprocessing

## Summary

Preprocessing of LC-MS/MS feature table and MS/MS spectral data within the msFeaST workflow so that feature MS/MS spectra become comparable across features before spectral similarity scoring and k-medoid clustering.

## When to use

Use when an LC-MS/MS untargeted metabolomics dataset (e.g. a Kathib et al. 2024 feature table with quantification, metadata, and MS/MS spectra) must be prepared for msFeaST spectral similarity computation.

## When NOT to use

- The goal is only to inspect pre-processed example files in the interactive dashboard — the README states only the msFeaST_Dashboard_bundle.html and a ready-made dashboard_data.json are needed, with no preprocessing.
- Input is already the msFeaST similarity/distance matrix or dashboard_data.json; this skill concerns the upstream preprocessing of quantification, metadata, and spectral data.
- Data are not LC-MS/MS untargeted metabolomics feature/spectral tables and cannot supply the quantification table, metadata table, and MS/MS spectra the pipeline requires.

## Inputs

- LC-MS/MS quantification table
- metadata table
- MS/MS spectral data (e.g. from the Kathib et al. example dataset bundled in notebooks/data)

## Outputs

- Processed MS/MS spectra per feature (processed data files used by the msFeaST spectral similarity module)
- Prepared feature set suitable for pairwise spectral similarity scoring and downstream k-medoid clustering / t-SNE

## How to apply

Within the msFeaST Python workflow, open the provided preprocessing notebook (e.g. preprocessing_mushroom_type_comparison.ipynb) alongside the pipeline notebook, point its filepath arguments to your quantification table, metadata table, and spectral data, and run the notebook to produce processed MS/MS spectra for each feature. The magenta-italics-marked cells flag the required user inputs, so update data filepath arguments to your own data file locations before running. The preprocessed MS/MS spectra are the input to the msFeaST spectral similarity computation stage, where the user-selected similarity score is applied over all pairwise feature comparisons, and the resulting distance matrix feeds k-medoid clustering and t-SNE embedding. Validate the preprocessing by confirming the resulting spectra are comparable across features and that the downstream similarity matrix is square and symmetric with one row/column per feature and self-similarity on the diagonal.

## Related tools

- **msFeaST** (Python/R/browser workflow whose preprocessing notebook processes the quantification table, metadata table, and spectral data into processed MS/MS spectra for spectral similarity scoring.) — https://github.com/kevinmildau/msFeaST

## Examples

```
conda activate msfeast_environment; jupyter-notebook notebooks/preprocessing_mushroom_type_comparison.ipynb notebooks/msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- Preprocessed MS/MS spectra are comparable across features so pairwise similarity scores are well defined.
- Downstream msFeaST spectral similarity stage yields a distance matrix that is square, symmetric, one row/column per feature, with self-similarity on the diagonal.
- Pipeline notebooks run to completion in the msfeast_environment conda environment (python=3.10, R 4.3.3) without requiring additional user edits beyond the marked filepaths.
- A JSON-format dashboard input file is produced by the jupyter-notebook pipeline and loads in the msFeaST interactive dashboard dataview tab.
- All required user-input cells (magenta italics) have been replaced with real data file locations.

## Limitations

- Workflow tested on macOS (macos-arm64) and expected to work identically on Linux; Windows support is still being worked on.
- rscript calls can install R packages into the wrong cached R path if the terminal is reused before conda R installation; close and reopen the terminal and re-activate msfeast_environment.
- Some terminals (e.g. vscode) default into specific conda environments, causing R/rscript path misalignment; verify with `type R` and `type rscript` that both sit within the conda environment.
- R dependencies (Matrix 1.6-5, survival 3.5-8, listenv 0.9.1, readr 2.1.5, tibble 3.2.1, dplyr 1.1.4, globaltest via BiocManager Bioconductor 3.18) must be installed at pinned versions; msFeaST will not run successfully if the R configuration check fails.
- Magenta-italic-highlighted cells in the preprocessing notebook require explicit user input of data filepaths — running with defaults will process the bundled example, not your data.

## Evidence

- [readme] These notebooks contain a complete example of quantification table, metadata table, and spectral data processing required for msFeaST, as well as a complete use-case example.: [evidence span withheld — anchor status: external_doc]
- [readme] To make use of your own data, change the data filepath arguments to your own data file location and run the pipeline.: [evidence span withheld — anchor status: external_doc]
- [abstract] The msFeaST workflow is freely available through https://github.com/kevinmildau/msFeaST: "The msFeaST workflow is freely available through https://github.com/kevinmildau/msFeaST and built to work on MacOS and Linux systems."
- [readme] The jupyter-notebook pipeline produces the a text file in json format that can be interactively explored in the interactive dashboard.: [evidence span withheld — anchor status: external_doc]
