---
name: lc-msms-feature-table-processing
description: Use when starting from raw LC-MS/MS feature abundance tables, sample
  group metadata, and MS/MS peak lists (e.g., the Pleurotus eryngii 0% vs 80% olive
  mill solid waste contrast) and needing a cleaned, msFeaST-ready data bundle.
when_to_use_negative:
- You only want to inspect pre-processed example files using the visual dashboard
  — then the msFeaST_Dashboard_bundle.html plus a ready-made .json file is sufficient
  and no dependencies need to be installed.
- Input is not LC-MS/MS untargeted metabolomics data with MS/MS spectra (msFeaST is
  built for LC-MS/MS untargeted metabolomics data).
- You are working on Windows — support is currently being worked on; the preprocessing/pipeline
  workflow has been tested on macOS and should work identically on Linux.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: msFeaST
  role: Python + R + browser workflow that pre-processes quantification/metadata/spectral
    data, computes spectral similarity (e.g., modified cosine score), runs k-medoids
    feature grouping, and emits dashboard JSON
  repo: https://github.com/kevinmildau/msFeaST
- name: globaltest
  role: R package used for feature group-wise statistical testing of k-medoids clusters
    against the sample-group contrast
- name: conda
  role: Manages the msfeast_environment with python=3.10 and conda-forge r-base=4.3
    for reproducible installation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae584
    title: Combined LC-MS/MS feature grouping, statistical prioritization, and interactive
      networking in msFeaST
  source_content_hash: sha256:3638057e2bcfb8e1a9dcb6f77468a09784625a6a73f0236be7df650099dbdd57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/msFeaST
  tools:
  - conda
  - globaltest
  - msFeaST
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-msms-feature-table-processing

## Summary

Pre-process LC-MS/MS untargeted metabolomics quantification tables, sample metadata, and MS/MS spectral peak lists into a JSON bundle compatible with the msFeaST interactive dashboard and downstream k-medoids feature grouping plus globaltest feature-set testing.

## When to use

Use when starting from raw LC-MS/MS feature abundance tables, sample group metadata, and MS/MS peak lists (e.g., the Pleurotus eryngii 0% vs 80% olive mill solid waste contrast) and needing a cleaned, msFeaST-ready data bundle.

## When NOT to use

- You only want to inspect pre-processed example files using the visual dashboard — then the msFeaST_Dashboard_bundle.html plus a ready-made .json file is sufficient and no dependencies need to be installed.
- Input is not LC-MS/MS untargeted metabolomics data with MS/MS spectra (msFeaST is built for LC-MS/MS untargeted metabolomics data).
- You are working on Windows — support is currently being worked on; the preprocessing/pipeline workflow has been tested on macOS and should work identically on Linux.

## Inputs

- LC-MS/MS feature abundance/quantification table
- sample metadata table with group labels (e.g., 0% vs 80% olive mill solid waste)
- MS/MS spectral peak lists per feature

## Outputs

- JSON data bundle for the interactive dashboard (e.g., dashboard_data.json)
- modified_cosine_stats_table.csv with cluster-wise P-values and per-feature log-2 fold changes
- spectral similarity-based feature groupings (k-medoids clusters)

## How to apply

Open the provided preprocessing and pipeline Jupyter notebooks (e.g., preprocessing_mushroom_type_comparison.ipynb followed by msfeast_pipeline_mushroom_type_comparison.ipynb) in the msfeast_environment and replace the default data filepath arguments with the locations of your own quantification table, metadata table, and spectral files. The notebooks perform the full pre-processing of the quantification table, metadata table, and spectral data required by msFeaST; magenta italic text in the notebooks marks required user input. The pipeline computes a pairwise spectral similarity matrix — here the modified cosine score for the 0% vs 80% olive mill solid waste contrast — and groups features into structurally similar clusters via k-medoids clustering, with the resulting clusters serving as a proxy for grouping structurally similar features. Group-wise statistical testing is then done with the globaltest package (installed at Bioconductor version 3.18, i.e., globaltest 5.56.0 under R 4.3.3), which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties. The notebook writes the combined cluster-wise P-values and per-feature log-2 fold changes (0% vs 80%) to modified_cosine_stats_table.csv and produces a JSON text file (e.g., dashboard_data.json under notebooks/data/omsw_pleurotus_ms2deepscore/) that can be loaded into the msFeaST_Dashboard_bundle.html for interactive exploration.

## Related tools

- **msFeaST** (Python + R + browser workflow that pre-processes quantification/metadata/spectral data, computes spectral similarity (e.g., modified cosine score), runs k-medoids feature grouping, and emits dashboard JSON) — https://github.com/kevinmildau/msFeaST
- **globaltest** (R package used for feature group-wise statistical testing of k-medoids clusters against the sample-group contrast)
- **conda** (Manages the msfeast_environment with python=3.10 and conda-forge r-base=4.3 for reproducible installation)

## Examples

```
conda activate msfeast_environment; jupyter-notebook  # open notebooks/preprocessing_mushroom_type_comparison.ipynb, set data filepaths, then run msfeast_pipeline_mushroom_type_comparison.ipynb
```

## Evaluation signals

- rscript -e "catch <- lapply(c('dplyr','tibble','readr','listenv','BiocManager','globaltest'), library, character.only=TRUE); sessionInfo()" must report R version 4.3.3 with globaltest_5.56.0, survival_3.5-8, listenv_0.9.1, readr_2.1.5, tibble_3.2.1, and dplyr_1.1.4 attached.
- The Jupyter notebook pipeline produces a text file in JSON format loadable into msFeaST_Dashboard_bundle.html; after loading, switching to the dataview tab shows the loaded data.
- The written modified_cosine_stats_table.csv must contain cluster-wise P-values from globaltest plus per-feature log-2 fold changes comparable to the deposited additional run reported in Section 3 of the article.
- For the illustrative Pleurotus eryngii 0% vs 80% olive mill solid waste example, the result must show many metabolites with elevated log-2 fold changes and a lower number with corresponding statistical enrichment patterns.

## Limitations

- The pre-processing and pipeline workflow has been tested on macOS and should work identically on Linux; Windows support is currently being worked on (untested, requires ANACONDA PROMPT).
- If rscript runs from the terminal before conda R is installed, the cached R path may be wrong, so packages can install to the wrong R — close the terminal after the pip install step, reopen it, and re-activate msfeast_environment before running the rscript installs.
- Some terminals (e.g., the vscode terminal) may default into specific conda environments and cause R path misalignment; check `type R` and `type rscript` both sit within the conda environment.
- Conda R package environments are not reliable for the required packages, so R packages are installed via remotes/BiocManager with pinned versions instead (Matrix 1.6-5, survival 3.5-8, listenv 0.9.1, readr 2.1.5, tibble 3.2.1, dplyr 1.1.4, globaltest 3.18).

## Evidence

- [readme] Notebook-driven preprocessing of quantification/metadata/spectral data: [evidence span withheld — anchor status: external_doc]
- [readme] Dashboard needs only JSON + bundle: [evidence span withheld — anchor status: external_doc]
- [readme] Pinned R dependency versions: [evidence span withheld — anchor status: external_doc]
- [readme] R sessionInfo check: [evidence span withheld — anchor status: external_doc]
- [results] Modified cosine + k-medoids + globaltest run on the 0% vs 80% OMSW contrast: [evidence span withheld — anchor status: not_found]
- [abstract] k-medoids clusters as structural proxies: "spectral similarity-based feature groupings generated using k-medoids clustering, where the resulting clusters serve as a proxy for grouping structurally similar features"
- [abstract] Why globaltest: "using the globaltest package, which provides high power to detect small concordant effects via joint modeling and reduced multiplicity adjustment penalties"
- [readme] Dashboard output type: [evidence span withheld — anchor status: external_doc]
