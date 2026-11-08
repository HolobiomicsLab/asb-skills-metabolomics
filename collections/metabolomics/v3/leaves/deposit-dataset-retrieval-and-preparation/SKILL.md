---
name: deposit-dataset-retrieval-and-preparation
description: Use when a task requires obtaining example principal coordinates and
  mapping files from a published data deposit (GigaScience Database, EMPeror FTP site)
  and staging them for QIIME/EMPeror ordination visualization.
when_to_use_negative:
- The dataset or visualization artifacts are already locally available and validated
  — this skill only covers deposit retrieval and staging, not the subsequent coloring
  or rendering steps.
- The environment runs on Windows or cannot provide Python 2.7 with QIIME (python
  libraries only), NumPy, BIOM 1.1.0 and PyCogent — the EMPeror command line interface
  is stated as OS X (10.6 and higher) and Linux only.
- The input is raw sequencing data rather than an already-computed principal coordinates
  file plus metadata mapping file; EMPeror consumes ordination output, not raw reads.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Command line interface consumes the retrieved principal coordinates and mapping
    files to produce the WebGL 3D PCoA visualization; deposit retrieval is validated
    by whether its CLI can load the staged files
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Defines the file formats being staged (principal coordinates file and metadata
    mapping file) and supplies the required python libraries
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:0fc5965a69c7bcadea63c28da6a10052b68441a703b8e05bace6f8b738480039
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# deposit-dataset-retrieval-and-preparation

## Summary

Retrieves the example PCoA and metadata mapping files supporting the EMPeror paper from their data deposits (GigaScience Database or the EMPeror FTP site) and prepares them so the EMPeror command line interface can build interactive 3D PCoA visualizations of the combined 5,740-sample / 120-metadata-column dataset.

## When to use

Use when a task requires obtaining example principal coordinates and mapping files from a published data deposit (GigaScience Database, EMPeror FTP site) and staging them for QIIME/EMPeror ordination visualization.

## When NOT to use

- The dataset or visualization artifacts are already locally available and validated — this skill only covers deposit retrieval and staging, not the subsequent coloring or rendering steps.
- The environment runs on Windows or cannot provide Python 2.7 with QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent — the EMPeror command line interface is stated as OS X (10.6 and higher) and Linux only.
- The input is raw sequencing data rather than an already-computed principal coordinates file plus metadata mapping file; EMPeror consumes ordination output, not raw reads.

## Inputs

- Deposit-resolved example files: QIIME principal coordinates file for the combined dataset of references [3, 20-22]
- Metadata mapping file with 120 metadata columns for 5,740 samples (including the age metadata field)
- Deposit URLs from the GigaScience Database [24] and EMPeror FTP site [25]

## Outputs

- Locally staged PCoA + mapping file pair ready for the EMPeror command line interface
- EMPeror-generated interactive WebGL 3D PCoA visualization (~26 MB bundle) loadable in Chrome
- Exported static rendering (e.g., figure_1b_age_coloring.png) of the metadata-colored plot

## How to apply

Download the example files — a QIIME principal coordinates file and a metadata mapping file for the combined dataset of references [3, 20-22] (5,740 samples, 120 metadata columns) — from the GigaScience Database or the EMPeror FTP site; note the article cites these deposits only by bracketed reference numbers ([24], [25]) and gives no archival DOI or accession IDs, so retrieval depends on resolving those references. Verify the downloaded bundle is compact and loadable: EMPeror output files are only 26 MB for 5,740 samples and 120 metadata columns (1.3% of the 1.85 GB KiNG equivalent), so files in the GB range indicate the wrong artifact was fetched. Confirm the environment meets the stated requirements — Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent — and that the OS is OS X (10.6 or higher) or Linux, since the command line interface is not supported on Windows. Prepare the mapping file so the metadata column of interest (e.g., age) is present as either a gradient or a categorical field, because EMPeror's controllers modify point coloring as a function of metadata; then pass the PCoA and mapping files to the EMPeror command line interface to produce the interactive WebGL 3D visualization for opening in Chrome.

## Related tools

- **EMPeror** (Command line interface consumes the retrieved principal coordinates and mapping files to produce the WebGL 3D PCoA visualization; deposit retrieval is validated by whether its CLI can load the staged files) — http://emperor.colorado.edu
- **QIIME** (Defines the file formats being staged (principal coordinates file and metadata mapping file) and supplies the required python libraries)

## Evaluation signals

- Downloaded PCoA and mapping files load successfully in the EMPeror command line interface and produce an interactive WebGL visualization openable in Chrome.
- The staged dataset matches the reported scale: 5,740 samples with 120 metadata columns (combined datasets of references 3, 20-22).
- File-size invariant: the EMPeror artifact is ~26 MB, i.e., ~1.3% of the 1.85 GB KiNG-equivalent files for the same data; GB-scale files indicate the wrong deposit artifact was retrieved.
- The mapping file retains the age metadata field in a form usable by both gradient-coloring and discrete/categorical-coloring controllers.
- Downstream export yields a loadable PNG (e.g., figure_1b_age_coloring.png) whose infant-vs-adult separation matches the published Figure 1B panels by expert visual comparison.

## Limitations

- No archival DOI or accession IDs are given for the supporting example files and data sets — deposits are cited only by bracketed references ([24], [25]), complicating automated retrieval.
- The command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated.
- The stack depends on Python 2.7 (end-of-life) and requires Chrome for the GUI, suggesting untested compatibility with other browsers and modern environments.

## Evidence

- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25].: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [abstract] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
