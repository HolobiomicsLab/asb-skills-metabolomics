---
name: extracted-ion-chromatogram-processing
description: Use when processing high-performance thin-layer chromatography coupled
  with high-resolution mass spectrometry data where extracted ion chromatograms must
  be clustered to highlight signal (ions of interest) over background ions.
when_to_use_negative:
- When the data are not HPTLC-HRMS derived — the method is framed for 'high-performance
  thin-layer chromatography coupled with high resolution mass spectrometry data'.
- When a fully automated, non-interactive background-removal workflow is required
  — the package explicitly 'launch a shiny app where the user can interactively analyse
  the data', so it is unsuitable for headless batch pipelines.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Runtime environment in which the eicCluster package and its Shiny app are
    installed and executed.
- name: shiny
  role: Framework backing the single function run_eicCluster() that launches the interactive
    app for analysing HPTLC-HRMS data.
- name: eicCluster
  role: The package implementing EIC clusterisation for background removal in HPTLC-HRMS
    data; installed from GitHub and invoked via eicCluster::run_eicCluster().
  repo: https://github.com/dimitrif/eicCluster
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.chroma.2018.09.050
    title: eicCluster software, an open-source in silico tool, and on-surface syntheses,
      an in situ concept, both exploited for signal highlighting in high-resolution
      mass spectrometry to ease structure elucidation in planar chromatography
  source_content_hash: sha256:59d68526c341329946ba1f0bf67f4baa5d7560495f6ec3c80f619f0128bbf8cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dimitrif/eicCluster
  tools:
  - R
  - RColorBrewer
  - Rtsne
  - eicCluster
  - readMzXmlData
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# extracted-ion-chromatogram-processing

## Summary

Clustering of extracted ion chromatograms (EICs) from HPTLC-HRMS data to separate ions of interest from background ions, provided as an interactive Shiny app that eases structure elucidation in planar chromatography.

## When to use

Use when processing high-performance thin-layer chromatography coupled with high-resolution mass spectrometry data where extracted ion chromatograms must be clustered to highlight signal (ions of interest) over background ions.

## When NOT to use

- When the data are not HPTLC-HRMS derived — the method is framed for 'high-performance thin-layer chromatography coupled with high resolution mass spectrometry data'.
- When a fully automated, non-interactive background-removal workflow is required — the package explicitly 'launch a shiny app where the user can interactively analyse the data', so it is unsuitable for headless batch pipelines.

## Inputs

- HPTLC-HRMS data containing extracted ion chromatograms (as analysed within the Shiny app)

## Outputs

- Partition of extracted ion chromatograms into ions of interest vs background ions (interactive clustering result in the Shiny app)

## How to apply

Install R and the eicCluster package from GitHub using devtools (install_github("dimitrif/eicCluster")). Invoke the package's single exported function run_eicCluster(), which launches a Shiny application; there the user interactively analyses the HPTLC-HRMS data by clusterising extracted ion chromatograms. The clusterisation partitions EICs into ions of interest versus background ions, which is the article's stated intent: to separate signal from background in HPTLC-HRMS data so that structure elucidation in planar chromatography becomes easier. The workflow is deliberately interactive, so the analyst inspects and guides the clustering rather than running a fixed parameterised script.

## Related tools

- **R** (Runtime environment in which the eicCluster package and its Shiny app are installed and executed.)
- **shiny** (Framework backing the single function run_eicCluster() that launches the interactive app for analysing HPTLC-HRMS data.)
- **eicCluster** (The package implementing EIC clusterisation for background removal in HPTLC-HRMS data; installed from GitHub and invoked via eicCluster::run_eicCluster().) — https://github.com/dimitrif/eicCluster

## Examples

```
install.packages("devtools"); library(devtools); install_github("dimitrif/eicCluster"); eicCluster::run_eicCluster()
```

## Evaluation signals

- The command eicCluster::run_eicCluster() executes without error after install_github("dimitrif/eicCluster") and a Shiny session starts.
- The app presents an interactive session in which the analyst can clusterise extracted ion chromatograms rather than emitting a static output file.
- The clustering separates EICs into ions of interest versus background ions for the HPTLC-HRMS track, matching the article's stated purpose of separating the ions of interest from the background ions.
- The result highlights signal to ease structure elucidation in planar chromatography, as described in the article's framing.

## Limitations

- The method is specific to HPTLC-HRMS data; no support for other acquisition modes is described.
- Analysis is interactive only — there is no described command-line or headless mode beyond launching the Shiny app.
- Installation from CRAN is not yet available ('Incomming'), so the GitHub route via devtools is required.

## Evidence

- [intro] Based on clusterisation of the extracted ion chromatograms, this method intends to separate the ions of interest from the background ions in high-performance thin-layer chromatography coupled with high resolution mass spectrometry data.: "Based on clusterisation of the extracted ion chromatograms, this method intends to separate the ions of interest from the background ions in high-performance thin-layer chromatography coupled with"
- [readme] The only function of the package launch a shiny app where the user can interactively analyse the data.: "The only function of the package launch a shiny app where the user can interactively analyse the data."
- [intro] Install R https://www.r-project.org/: "Install R https://www.r-project.org/"
- [intro] install_github("dimitrif/eicCluster"): "install_github("dimitrif/eicCluster")"
- [readme] eicCluster::run_eicCluster(): "eicCluster::run_eicCluster()"
