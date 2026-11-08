---
name: shiny-application-launcher-implementation
description: Use when building an R package whose sole user-facing entry point must
  start an interactive Shiny app, e.g. exposing run_eicCluster() from an installed
  GitHub package.
when_to_use_negative:
- When a non-interactive, scriptable command-line pipeline is needed — the launcher
  explicitly opens a shiny app for the user to interactively analyse the data.
- When the data are not HPTLC-HRMS extracted ion chromatograms requiring background
  clusterisation; the app's scope is separating ions of interest from background ions
  in that modality.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Runtime language in which run_eicCluster is written and from which the Shiny
    app is launched; users install R from https://www.r-project.org/ per the README.
  repo: https://www.r-project.org/
- name: eicCluster
  role: The R package containing the exported run_eicCluster(port) launcher that starts
    the Shiny app for interactive background removal in HPTLC-HRMS data.
  repo: github.com/dimitrif/eicCluster
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.chroma.2018.09.050
    title: eicCluster software, an open-source in silico tool, and on-surface syntheses,
      an in situ concept, both exploited for signal highlighting in high-resolution
      mass spectrometry to ease structure elucidation in planar chromatography
  source_content_hash: sha256:3626bfac74d57ca5018222c952136edece4887095e192220a93b6fc459105090
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - eicCluster
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# shiny-application-launcher-implementation

## Summary

Implement and expose a single exported R entry-point function (run_eicCluster) that launches a Shiny application for interactive background-ion removal in HPTLC-HRMS data via clusterisation of extracted ion chromatograms.

## When to use

Use when building an R package whose sole user-facing entry point must start an interactive Shiny app, e.g. exposing run_eicCluster() from an installed GitHub package.

## When NOT to use

- When a non-interactive, scriptable command-line pipeline is needed — the launcher explicitly opens a shiny app for the user to interactively analyse the data.
- When the data are not HPTLC-HRMS extracted ion chromatograms requiring background clusterisation; the app's scope is separating ions of interest from background ions in that modality.

## Inputs

- R environment (installed via https://www.r-project.org/) with devtools for GitHub installation
- HPTLC-HRMS data whose extracted ion chromatograms will be clusterised in the interactive app

## Outputs

- A running Shiny application session on the requested port where the user can interactively analyse HPTLC-HRMS data and separate ions of interest from background ions

## How to apply

Write a function run_eicCluster with a formal port argument, so users can control the listening port passed to shiny::runApp. Inside the launcher, assemble the Shiny UI and server that use shiny, shinyBS, and shinydashboard to provide the interactive analysis interface for clusterising extracted ion chromatograms (EICs) from high-performance thin-layer chromatography coupled with high-resolution mass spectrometry (HPTLC-HRMS). Declare run_eicCluster as the package's only exported symbol in the NAMESPACE, keeping all internal app components unexported. Install the package from GitHub with devtools::install_github("dimitrif/eicCluster") after installing R from https://www.r-project.org/. The launcher rationale is that the README documents a one-line launch command, so the function must encapsulate app startup rather than requiring users to source UI/server files manually.

## Related tools

- **R** (Runtime language in which run_eicCluster is written and from which the Shiny app is launched; users install R from https://www.r-project.org/ per the README.) — https://www.r-project.org/
- **eicCluster** (The R package containing the exported run_eicCluster(port) launcher that starts the Shiny app for interactive background removal in HPTLC-HRMS data.) — github.com/dimitrif/eicCluster

## Examples

```
eicCluster::run_eicCluster()
```

## Evaluation signals

- Loading the installed package and running eicCluster::run_eicCluster() from an R console starts the Shiny application without error, matching the README launch command.
- run_eicCluster is the package's only exported symbol; find.package("eicCluster") then checkNamespaceExports confirms no other exported functions.
- The launched app renders the shinydashboard/shinyBS interface and allows interactive analysis of the HPTLC-HRMS data as described in the README.
- The port argument is honoured: passing a distinct port causes shiny::runApp to serve the app on that port.
- Calling the function after devtools::install_github("dimitrif/eicCluster") reproduces the README's documented one-command launch flow.

## Limitations

- CRAN availability was incomplete at writing time — the README's 'From cran' section only states 'Incomming', so GitHub installation via devtools was the supported path.
- The launcher is a thin wrapper: all interactive analysis logic lives in the app's UI/server, so testing only the launcher does not validate clustering behaviour.
- If the port is occupied or unspecified in a restricted environment, the app may fail to start; the port formal argument is the only user-controllable launch parameter.

## Evidence

- [readme] The only function of the package launch a shiny app where the user can interactively analyse the data.: "The only function of the package launch a shiny app where the user can interactively analyse the data."
