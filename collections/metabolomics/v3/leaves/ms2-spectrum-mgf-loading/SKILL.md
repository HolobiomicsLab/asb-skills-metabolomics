---
name: ms2-spectrum-mgf-loading
description: Use when starting a mineMS2 analysis from raw MGF spectra (e.g. pnordicum_ms2_spectra.mgf)
  plus a TSV metadata file, before discretizeMzDifferences.
when_to_use_negative:
- Data are already an ms2Lib object or already-fragmentation-graphed spectra — skip
  straight to discretizeMzDifferences/pattern mining
- Input is an mzML or raw vendor file not in MGF format — convert to MGF first or
  use an alternative loader
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mineMS2
  role: R package whose ms2Lib function loads the MGF spectra and metadata into an
    ms2Lib object
  repo: https://github.com/odisce/mineMS2
- name: R
  role: runtime environment for installing and running mineMS2 (devtools::install_github)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:aac11630c5eb2e69ebbb2de946eee15596236c9577c75c4aaf6bda35d1298abe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - R
  - mineMS2
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ms2-spectrum-mgf-loading

## Summary

Ingests an MGF file of MS/MS spectra with supplementary metadata into a mineMS2 ms2Lib object, the mandatory first stage before fragmentation-graph construction and pattern mining.

## When to use

Use when starting a mineMS2 analysis from raw MGF spectra (e.g. pnordicum_ms2_spectra.mgf) plus a TSV metadata file, before discretizeMzDifferences.

## When NOT to use

- Data are already an ms2Lib object or already-fragmentation-graphed spectra — skip straight to discretizeMzDifferences/pattern mining
- Input is an mzML or raw vendor file not in MGF format — convert to MGF first or use an alternative loader

## Inputs

- MGF file of MS/MS spectra (e.g. pnordicum_ms2_spectra.mgf)
- TSV metadata table (e.g. pnordicum_ms2_info.tsv)
- R with devtools and the mineMS2 package installed from GitHub

## Outputs

- ms2Lib R object (pnordicum_m2l) holding spectra plus metadata, ready for fragmentation-graph construction via discretizeMzDifferences

## How to apply

Read the MGF file of MS/MS spectra together with a supplementary metadata table (e.g. pnordicum_ms2_info.tsv loaded via system.file("dataset/pnordicum_ms2_info.tsv", package="mineMS2")) and call ms2Lib(mgf_file, suppInfos = metadata_df) from the mineMS2 R package (install via devtools::install_github("odisce/mineMS2")). This converts each spectrum into an ms2Lib object that retains the spectra and their metadata, which is then passed to discretizeMzDifferences with dmz=0.007, ppm=15, and maxFrags=15 (the 15 most intense fragment ions per spectrum) to build fragmentation graphs. Install devtools first with install.packages("devtools") if needed.

## Related tools

- **mineMS2** (R package whose ms2Lib function loads the MGF spectra and metadata into an ms2Lib object) — https://github.com/odisce/mineMS2
- **R** (runtime environment for installing and running mineMS2 (devtools::install_github))

## Examples

```
pnordicum_mgf.c <- system.file("dataset/pnordicum_ms2_spectra.mgf", package = "mineMS2"); pnordicum_metadata.df <- read.table(system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"), header = TRUE, sep = "\t"); pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)
```

## Evaluation signals

- ms2Lib() returns a non-null ms2Lib object containing the 51 Penicillium nordicum spectra plus metadata columns from the TSV
- Downstream discretizeMzDifferences on this object accepts dmz=0.007, ppm=15, maxFrags=15 without error
- The loaded object matches the metadata keys in pnordicum_ms2_info.tsv (one row per spectrum)
- Installation completes without devtools::install_github errors and the vignette dataset is discoverable via system.file

## Limitations

- Only MGF format is documented in the mineMS2 workflow; other input formats are not supported by ms2Lib
- The number of spectra (51 for the included Penicillium nordicum dataset) is small; mineMS2 processes up to 3,000 spectra in a few hours on a standard laptop but larger collections may be slow
- No software version number is stated for mineMS2; the package is installed from the GitHub master branch which may change

## Evidence

- [intro] Article introduction — motivation for loading MS/MS collections into a structured representation: "we propose an innovative combinatorial strategy to extract fragmentation patterns from MS/MS spectra collections, based on a representation of the MS/MS spectrum as a Directed Acyclic Graph (DAG)"
- [methods] Article methods — description of the ms2Lib stage in the mineMS2 package: "The described algorithm was written in C++ using Boost, and wrapped with Rcpp into the mineMS2 R package"
- [readme] README — installation and loading instructions: [evidence span withheld — anchor status: external_doc]
- [readme] README — dataset description used in the vignettes: [evidence span withheld — anchor status: not_found]
- [supplementary] Article supplementary — R code showing ms2Lib call with metadata: "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")
pnordicum.m2l <- ms2Lib(pnordicum_mgf.c, suppInfos = pnordicum_metadata.df)"
- [supplementary] Source card — concrete invocation grounded in the paper: [evidence span withheld — anchor status: not_found]
