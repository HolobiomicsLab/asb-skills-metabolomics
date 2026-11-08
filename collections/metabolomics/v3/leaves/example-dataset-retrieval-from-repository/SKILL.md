---
name: example-dataset-retrieval-from-repository
description: Reference for obtaining the ready-made example LC-MS/MS dataset (processed
  MS/MS spectra, quantification table, and metadata table) shipped with the msFeaST
  GitHub repository, e.g. when setting up or validating the msFeaST workflow before
  running it on user-owned data.
when_to_use_negative:
- The user already possesses their own complete quantification table, metadata table,
  and MS/MS spectra; swapping in user data is the intended path once the example run
  has been validated.
- Only dashboard inspection is desired without running the Python/R pipeline (just
  download msFeaST_Dashboard_bundle.html and the JSON data, no conda environment needed).
- Windows support is still being worked on and the commands below are macOS/Linux-specific;
  do not use this skill to obtain Windows-compatible setup instructions.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: msFeaST
  role: Provides the repository and the bundled example dataset (quantification table,
    metadata table, processed MS/MS spectra) that this skill retrieves; the example
    files are the canonical test fixtures for the msFeaST workflow.
  repo: https://github.com/kevinmildau/msFeaST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:1ece1ffa8d97a545847723e0f1eb051d98098d613d54781997991090b24d8ffa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - msFeaST
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# example-dataset-retrieval-from-repository

## Summary

Retrieves the bundled example dataset (Kathib et al. 2024 LC-MS/MS feature table with processed MS/MS spectra, quantification table, and metadata table) from the msFeaST GitHub repository so that the pipeline can be exercised end-to-end before applying it to user data.

## When to use

Reference for obtaining the ready-made example LC-MS/MS dataset (processed MS/MS spectra, quantification table, and metadata table) shipped with the msFeaST GitHub repository, e.g. when setting up or validating the msFeaST workflow before running it on user-owned data.

## When NOT to use

- The user already possesses their own complete quantification table, metadata table, and MS/MS spectra; swapping in user data is the intended path once the example run has been validated.
- Only dashboard inspection is desired without running the Python/R pipeline (just download msFeaST_Dashboard_bundle.html and the JSON data, no conda environment needed).
- Windows support is still being worked on and the commands below are macOS/Linux-specific; do not use this skill to obtain Windows-compatible setup instructions.

## Inputs

- msFeaST GitHub repository clone (https://github.com/kevinmildau/msFeaST)
- notebooks/data directory containing ready-made example files
- msFeaST_Dashboard_bundle.html (for validating the loaded example data in a browser)

## Outputs

- Kathib et al. (2024) LC-MS/MS feature table with processed MS/MS spectra (spectral_similarity_matrix.csv input)
- dashboard_data.json file consumed by the interactive dashboard
- A populated conda environment + Jupyter notebook session ready to run the full pipeline on the example data

## How to apply

Clone the msFeaST repository at https://github.com/kevinmildau/msFeaST and locate the pre-processed example files referenced in the README: the quantification table, metadata table, and processed MS/MS spectra files (e.g. notebooks/data/omsw_pleurotus_ms2deepscore/dashboard_data.json for the dashboard bundle). Verify the example data is complete by loading the pre-processing notebook (preprocessing_mushroom_type_comparison.ipynb) which 'contain[s] a complete example of quantification table, metadata table, and spectral data processing required for msFeaST'. These bundled files provide the Kathib et al. (2024) LC-MS/MS feature table whose MS/MS spectra feed the pairwise spectral-similarity computation stage; they serve as the canonical test fixture for the k-medoid clustering, globaltest feature-set testing, and interactive dashboard visualization. Validation is done by loading the files and confirming the dashboard bundle (msFeaST_Dashboard_bundle.html) can ingest the JSON data without errors, which is the documented acceptance test for the example dataset.

## Related tools

- **msFeaST** (Provides the repository and the bundled example dataset (quantification table, metadata table, processed MS/MS spectra) that this skill retrieves; the example files are the canonical test fixtures for the msFeaST workflow.) — https://github.com/kevinmildau/msFeaST

## Examples

```
git clone https://github.com/kevinmildau/msFeaST && conda create --name msfeast_environment python=3.10 && conda activate msfeast_environment && pip install "git+https://github.com/kevinmildau/msfeast.git" && jupyter-notebook notebooks/preprocessing_mushroom_type_comparison.ipynb
```

## Evaluation signals

- The cloned repository contains the notebooks/data directory with the ready-made example files referenced by the README (dashboard_data.json under notebooks/data/omsw_pleurotus_ms2deepscore/).
- Opening preprocessing_mushroom_type_comparison.ipynb and msfeast_pipeline_mushroom_type_comparison.ipynb succeeds without missing-file errors, confirming the bundled example data is complete.
- The msFeaST_Dashboard_bundle.html, when opened in a desktop browser (firefox/chrome/edge/safari), loads the dashboard_data.json and shows data in the dataview tab — the README's documented acceptance test.
- The conda environment (msfeast_environment, python 3.10, R 4.3.3) can be activated and `rscript -e "catch <- lapply(c('dplyr','tibble','readr','listenv','BiocManager','globaltest'), library, character.only = TRUE); sessionInfo()"` returns the expected session info (globaltest_5.56.0, survival_3.5-8, etc.), confirming the dependency stack needed to consume the example dataset.
- The retrieved example feature table yields a square, symmetric similarity matrix with 1.0 on the diagonal when fed to the downstream spectral-similarity computation, confirming the dataset matches the schema expected by msFeaST.

## Limitations

- msFeaST's pre-processing and pipeline workflow has been tested on macOS and Linux; Windows support is still being worked on, so example dataset retrieval on Windows may require the Anaconda Prompt and is untested.
- R package installation inside the conda environment is currently done with remotes/BiocManager rather than conda because 'the conda R package environment is not working reliably for the required packages yet'; a stale rscript path can cause packages to install into the wrong R location.
- The interactive dashboard bundle works only in desktop browsers (firefox, chrome, edge, safari) and requires the separately downloaded msFeaST_Dashboard_bundle.html.

## Evidence

- [readme] README quickstart section: [evidence span withheld — anchor status: external_doc]
- [readme] README example notebook description: [evidence span withheld — anchor status: external_doc]
- [readme] README installation rationale: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract msFeaST scope statement: "Here, we present msFeaST (mass spectral Feature Set Testing), a feature-set testing and visualization workflow for LC-MS/MS untargeted metabolomics data."
- [readme] README dashboard OS support statement: [evidence span withheld — anchor status: external_doc]
