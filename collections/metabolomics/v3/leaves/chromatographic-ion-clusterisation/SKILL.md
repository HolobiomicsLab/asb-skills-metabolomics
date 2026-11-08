---
name: chromatographic-ion-clusterisation
description: Use when analysing HPTLC coupled with high-resolution mass spectrometry
  data (e.g. mzXML-derived extracted ion chromatograms) and you need to partition
  ions of interest from background ions before structure elucidation.
when_to_use_negative:
- The input is not HPTLC-HRMS data (e.g. bulk LC-MS/MS or untargeted metabolomics
  feature tables) — the method is framed specifically for high-performance thin-layer
  chromatography coupled with high-resolution mass spectrometry.
- You need a fully automated, non-interactive batch pipeline — the sole function of
  the package launches a Shiny app 'where the user can interactively analyse the data',
  so it cannot run unattended.
- mzXML-derived EICs have not yet been generated and you lack the upstream readMzXmlData/Rtsne
  tooling — the skill presupposes extracted ion chromatograms are already available
  for clusterisation.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: eicCluster
  role: R/Shiny package whose exported run_eicCluster() launches the interactive app
    to clusterise extracted ion chromatograms and separate ions of interest from background
    ions
  repo: https://github.com/OfficeChromatography/eicCluster
- name: R
  role: Runtime environment required to install and run the eicCluster Shiny app
  repo: https://www.r-project.org/
- name: devtools
  role: Used to install eicCluster from GitHub (install_github("dimitrif/eicCluster"))
- name: readMzXmlData
  role: 'Per source-card workflow: loads mzXML-format HPTLC-HRMS spectral data from
    which extracted ion chromatograms are built'
- name: Rtsne
  role: 'Per source-card workflow: computes dimensionality-reduced embedding of EIC
    features prior to clusterisation'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.chroma.2018.09.050
    title: eicCluster software, an open-source in silico tool, and on-surface syntheses,
      an in situ concept, both exploited for signal highlighting in high-resolution
      mass spectrometry to ease structure elucidation in planar chromatography
  source_content_hash: sha256:d311ea172ff2c01f40dc5e820c88759b1f33f444e9f3b2453bb4f84a0654edf4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OfficeChromatography/eicCluster
  tools:
  - R
  - RColorBrewer
  - Rtsne
  - devtools
  - eicCluster
  - readMzXmlData
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# chromatographic-ion-clusterisation

## Summary

Interactive R/Shiny workflow (eicCluster) that separates ions of interest from background ions in HPTLC-HRMS data by clusterising the extracted ion chromatograms, easing structure elucidation in planar chromatography.

## When to use

Use when analysing HPTLC coupled with high-resolution mass spectrometry data (e.g. mzXML-derived extracted ion chromatograms) and you need to partition ions of interest from background ions before structure elucidation.

## When NOT to use

- The input is not HPTLC-HRMS data (e.g. bulk LC-MS/MS or untargeted metabolomics feature tables) — the method is framed specifically for high-performance thin-layer chromatography coupled with high-resolution mass spectrometry.
- You need a fully automated, non-interactive batch pipeline — the sole function of the package launches a Shiny app 'where the user can interactively analyse the data', so it cannot run unattended.
- mzXML-derived EICs have not yet been generated and you lack the upstream readMzXmlData/Rtsne tooling — the skill presupposes extracted ion chromatograms are already available for clusterisation.

## Inputs

- HPTLC-HRMS data (extracted ion chromatograms derived from high-resolution mass spectrometry of high-performance thin-layer chromatography runs)
- mzXML-format raw spectral files (per task card workflow: readMzXmlData-derived EICs)

## Outputs

- Interactive Shiny session visualising clusters of extracted ion chromatograms, partitioned into ions of interest vs background ions
- Per-ion cluster/interest assignment table (eic_cluster_assignments.csv per source card)

## How to apply

Load the HPTLC-HRMS-derived extracted ion chromatograms (e.g. from mzXML spectral data) into R, then launch the eicCluster Shiny application with eicCluster::run_eicCluster(), which is the only exported function of the package and lets the user interactively analyse the data. In the app, clusterise the extracted ion chromatograms so that clusters corresponding to ions of interest can be separated from background-ion clusters; the clusterisation basis is explicitly 'to separate the ions of interest from the background ions' per the authors. Install via devtools with install_github("dimitrif/eicCluster") after installing R from https://www.r-project.org/. Review the interactive clustering output manually to confirm that the highlighted clusters correspond to analyte signal rather than background before proceeding to structure elucidation in planar chromatography.

## Related tools

- **eicCluster** (R/Shiny package whose exported run_eicCluster() launches the interactive app to clusterise extracted ion chromatograms and separate ions of interest from background ions) — https://github.com/OfficeChromatography/eicCluster
- **R** (Runtime environment required to install and run the eicCluster Shiny app) — https://www.r-project.org/
- **devtools** (Used to install eicCluster from GitHub (install_github("dimitrif/eicCluster")))
- **readMzXmlData** (Per source-card workflow: loads mzXML-format HPTLC-HRMS spectral data from which extracted ion chromatograms are built)
- **Rtsne** (Per source-card workflow: computes dimensionality-reduced embedding of EIC features prior to clusterisation)

## Examples

```
install.packages("devtools"); library(devtools); install_github("dimitrif/eicCluster"); eicCluster::run_eicCluster()
```

## Evaluation signals

- eicCluster::run_eicCluster() starts a Shiny session without error after install_github("dimitrif/eicCluster").
- The displayed clustering partitions extracted ion chromatograms into a cluster labelled as ions of interest and clusters identified as background ions.
- Per-ion assignment output (eic_cluster_assignments.csv) exists, is valid CSV, contains one row per ion with a cluster/interest label field.
- The separation of ions of interest from background ions is confirmed by expert review of the clustering result (per source-card validation criteria).
- The only exported function of the package (run_eicCluster) is the entry point used — no other API surface is expected.

## Limitations

- The method is tied to interactive analysis: 'The only function of the package launch a shiny app where the user can interactively analyse the data', so it is not scriptable headless.
- As of the README, CRAN installation was still pending ('From cran — Incomming'), so the package had to be installed from GitHub.
- The README/record notes 'No changelog found', so version history and reproducibility tracking are limited.
- Licensing of the OfficeChromatography/eicCluster repository is recorded as NOASSERTION, so reuse/redistribution terms are not established.

## Evidence

- [intro] Clusterisation separates ions of interest from background ions: "Based on clusterisation of the extracted ion chromatograms, this method intends to separate the ions of interest from the background ions in high-performance thin-layer chromatography coupled with"
- [intro] Interactive Shiny entry point: "The only function of the package launch a shiny app where the user can interactively analyse the data."
- [readme] GitHub install command: "install_github("dimitrif/eicCluster")"
- [readme] Invocation of the app: "Then, run this command to launch the application
```r
eicCluster::run_eicCluster()
```"
- [readme] App purpose: "This repository contain a shiny app to remove background in HPTLC-HRMS data."
