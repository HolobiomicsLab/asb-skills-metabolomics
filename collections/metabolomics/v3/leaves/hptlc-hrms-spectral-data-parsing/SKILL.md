---
name: hptlc-hrms-spectral-data-parsing
description: Use when you have mzXML-format spectral files from HPTLC-HRMS experiments
  and need to load the raw spectra into R as the first step before extracted ion chromatogram
  (EIC) clustering or other downstream analysis.
when_to_use_negative:
- The input data is already in a parsed R representation (e.g. an existing EIC feature
  table); the parsing step would be redundant.
- The spectra come from LC-MS or direct infusion rather than HPTLC-HRMS planar chromatography;
  eicCluster's workflow is designed for HPTLC-HRMS data specifically.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: readMzXmlData
  role: R package used to read the mzXML-format HPTLC-HRMS spectral files into R objects
    before EIC construction
- name: eicCluster
  role: Downstream package whose shiny app consumes the parsed HPTLC-HRMS data for
    interactive clusterisation of extracted ion chromatograms
  repo: github.com/dimitrif/eicCluster
- name: devtools
  role: R package used to install eicCluster from GitHub so the parsing/analysis toolchain
    is available
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.chroma.2018.09.050
    title: eicCluster software, an open-source in silico tool, and on-surface syntheses,
      an in situ concept, both exploited for signal highlighting in high-resolution
      mass spectrometry to ease structure elucidation in planar chromatography
  source_content_hash: sha256:dbd8b9f83fe17f6b35f8aae2a05b729625097bb23c23d114582ec28c41ffb91e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - RColorBrewer
  - Rtsne
  - devtools
  - eicCluster
  - readMzXmlData
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hptlc-hrms-spectral-data-parsing

## Summary

Load HPTLC-HRMS spectral data in mzXML format into R via readMzXmlData so the raw high-resolution mass spectra of a thin-layer chromatographic run can be fed into the eicCluster interactive analysis.

## When to use

Use when you have mzXML-format spectral files from HPTLC-HRMS experiments and need to load the raw spectra into R as the first step before extracted ion chromatogram (EIC) clustering or other downstream analysis.

## When NOT to use

- The input data is already in a parsed R representation (e.g. an existing EIC feature table); the parsing step would be redundant.
- The spectra come from LC-MS or direct infusion rather than HPTLC-HRMS planar chromatography; eicCluster's workflow is designed for HPTLC-HRMS data specifically.

## Inputs

- HPTLC-HRMS spectral data in mzXML format
- R session with readMzXmlData installed

## Outputs

- R objects holding parsed mzXML spectra (peak lists / mass-charge-retention information per scan) suitable for EIC feature construction
- Input to the eicCluster shiny app for interactive clusterisation

## How to apply

Acquire the HPTLC-HRMS raw spectral data in mzXML format (the article's workflow starts by loading this with the readMzXmlData package in R). Install readMzXmlData alongside the eicCluster package (installed from github.com/dimitrif/eicCluster via devtools) and call its mzXML reading functions to parse the mzXML into R objects. The parsed spectra serve as the input for building EIC features per detected ion, which are then reduced with t-SNE (Rtsne or tsne) and clusterised to separate ions of interest from background ions. This parsing step is foundational: without correctly reading the mzXML into R, the eicCluster shiny app has no HPTLC-HRMS data to display interactively. The reading functions handle the file conversion so the user can launch the shiny application and proceed to interactive analysis.

## Related tools

- **readMzXmlData** (R package used to read the mzXML-format HPTLC-HRMS spectral files into R objects before EIC construction)
- **eicCluster** (Downstream package whose shiny app consumes the parsed HPTLC-HRMS data for interactive clusterisation of extracted ion chromatograms) — github.com/dimitrif/eicCluster
- **devtools** (R package used to install eicCluster from GitHub so the parsing/analysis toolchain is available)

## Examples

```
install.packages("devtools"); library(devtools); install_github("dimitrif/eicCluster"); eicCluster::run_eicCluster()
```

## Evaluation signals

- The mzXML file is read without error and yields R objects containing mass/charge and retention (track) information per scan.
- The parsed data can be passed into the eicCluster workflow such that EIC features can be constructed for each detected ion.
- The resulting eicCluster shiny app opens and displays the HPTLC-HRMS data for interactive clusterisation.
- Output eic_cluster_assignments.csv (downstream) is a valid CSV with one row per ion and a cluster/interest label, confirming the parsed input fed the pipeline correctly.

## Limitations

- The parsing depends on the eicCluster package and readMzXmlData being installed; installation is documented as coming 'from github' via devtools with CRAN availability noted as 'Incomming', so availability on CRAN was not yet guaranteed at writing.
- Only mzXML format is explicitly supported in the documented workflow; other spectral formats would need conversion beforehand.
- The reading step alone does not separate ions of interest from background — the actual discrimination happens later in the t-SNE + clusterisation steps.

## Evidence

- [other] Workflow step 1: Load the HPTLC-HRMS-derived extracted ion chromatogram data (mzXML format) with the readMzXmlData package in R.: [evidence span withheld — anchor status: not_found]
- [readme] README installation section: "This repository contain a shiny app to remove background in HPTLC-HRMS data."
- [readme] README installation instructions for devtools/eicCluster: "install.packages("devtools")
library(devtools)
install_github("dimitrif/eicCluster")"
