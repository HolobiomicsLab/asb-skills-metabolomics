---
name: ion-of-interest-vs-background-partitioning
description: Use when HPTLC-HRMS data contains many EICs mixing analyte ions with
  background ions and the goal is to partition ion tracks into interesting vs background
  clusters before structure elucidation.
when_to_use_negative:
- When the input is already a curated feature table whose ions have already been partitioned
  into interesting vs background clusters — this skill targets raw EIC-level partitioning
  from HPTLC-HRMS mzXML data.
- When the data does not come from HPTLC-HRMS (e.g. standard LC-MS/MS feature lists
  without planar chromatography tracks), as the EIC clusterisation and background-removal
  rationale is specific to planar chromatography signal highlighting.
- When a fully automated, non-interactive clustering is required without expert visual
  review, since run_eicCluster() explicitly launches an interactive Shiny app for
  user analysis.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Runtime environment in which the eicCluster package and Shiny app are installed
    and executed for interactive HPTLC-HRMS analysis.
  repo: https://www.r-project.org/
- name: readMzXmlData
  role: Reads the mzXML HPTLC-HRMS raw data so EIC features can be built per detected
    ion prior to clusterisation.
- name: Rtsne
  role: Computes the dimensionality-reduced embedding of EIC features used for clusterisation
    to separate interesting from background ions.
- name: tsne
  role: Alternative implementation for the t-SNE embedding of EIC features if Rtsne
    is unavailable.
- name: shiny
  role: Provides the interactive app framework launched by run_eicCluster() where
    the user interactively analyses the HPTLC-HRMS EIC clusters.
- name: RColorBrewer
  role: Supplies colour palettes for visualising cluster assignment of EICs in the
    Shiny app.
- name: eicCluster
  role: The package exporting run_eicCluster(); encapsulates the EIC clusterisation
    workflow that separates ions of interest from background ions.
  repo: https://github.com/dimitrif/eicCluster
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.chroma.2018.09.050
    title: eicCluster software, an open-source in silico tool, and on-surface syntheses,
      an in situ concept, both exploited for signal highlighting in high-resolution
      mass spectrometry to ease structure elucidation in planar chromatography
  source_content_hash: sha256:cfdfeac878dc2ce2a52adcf65f595fb89a0f01a88c20168ee35b871619415389
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - RColorBrewer
  - Rtsne
  - eicCluster
  - readMzXmlData
  - shiny
  - tsne
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ion-of-interest-vs-background-partitioning

## Summary

Clusters extracted ion chromatograms (EICs) from HPTLC-HRMS runs so that ions carrying the analyte signal of interest can be visually and computationally separated from background ions, easing structure elucidation in planar chromatography.

## When to use

Use when HPTLC-HRMS data contains many EICs mixing analyte ions with background ions and the goal is to partition ion tracks into interesting vs background clusters before structure elucidation.

## When NOT to use

- When the input is already a curated feature table whose ions have already been partitioned into interesting vs background clusters — this skill targets raw EIC-level partitioning from HPTLC-HRMS mzXML data.
- When the data does not come from HPTLC-HRMS (e.g. standard LC-MS/MS feature lists without planar chromatography tracks), as the EIC clusterisation and background-removal rationale is specific to planar chromatography signal highlighting.
- When a fully automated, non-interactive clustering is required without expert visual review, since run_eicCluster() explicitly launches an interactive Shiny app for user analysis.

## Inputs

- HPTLC-HRMS raw spectral data in mzXML format (extracted ion chromatogram tracks per detected ion)
- Detected ion list derived from HPTLC-HRMS runs

## Outputs

- Per-ion assignment of cluster membership, separating interesting ions from background ions
- Dimensionality-reduced (t-SNE) embedding of EIC features used for clusterisation
- Interactive visualisation of clustered EICs in the Shiny app for expert review

## How to apply

Load the HPTLC-HRMS-derived EIC data (mzXML format, read with readMzXmlData in R), then build features per extracted ion chromatogram. Run the exported run_eicCluster() function, which launches a Shiny app for interactive analysis: reduce the EIC feature space via t-SNE embedding (Rtsne, with tsne as an alternative implementation) and clusterise the embedded features so that ions of interest separate from background ions in the high-performance thin-layer chromatography coupled with high resolution mass spectrometry data. The user interacts with the app to assign each EIC/track to a cluster, giving a label of interesting vs background. The separation is judged by whether clusters of analyte ions stand apart from background ions in the reduced space, supporting subsequent structure elucidation in planar chromatography.

## Related tools

- **R** (Runtime environment in which the eicCluster package and Shiny app are installed and executed for interactive HPTLC-HRMS analysis.) — https://www.r-project.org/
- **readMzXmlData** (Reads the mzXML HPTLC-HRMS raw data so EIC features can be built per detected ion prior to clusterisation.)
- **Rtsne** (Computes the dimensionality-reduced embedding of EIC features used for clusterisation to separate interesting from background ions.)
- **tsne** (Alternative implementation for the t-SNE embedding of EIC features if Rtsne is unavailable.)
- **shiny** (Provides the interactive app framework launched by run_eicCluster() where the user interactively analyses the HPTLC-HRMS EIC clusters.)
- **RColorBrewer** (Supplies colour palettes for visualising cluster assignment of EICs in the Shiny app.)
- **eicCluster** (The package exporting run_eicCluster(); encapsulates the EIC clusterisation workflow that separates ions of interest from background ions.) — https://github.com/dimitrif/eicCluster

## Examples

```
install.packages("devtools"); library(devtools); install_github("dimitrif/eicCluster"); eicCluster::run_eicCluster()
```

## Evaluation signals

- Confirm that the exported entry point run_eicCluster() exists and, when called, launches the Shiny app rather than exiting silently.
- Verify that the mzXML HPTLC-HRMS data is successfully read via readMzXmlData before any EIC features are built.
- Check that t-SNE embedding (Rtsne or tsne) produces a reduced-dimension representation of the EIC features suitable for cluster separation.
- Confirm that the clusterisation result partitions ions into an interesting group and a background group, with analyte ions visually distinct from background ions in the app.
- Validate that expert review of the interactive clustering agrees with expected analyte tracks in the HPTLC plate.

## Limitations

- Installation from CRAN is not yet available ('Incomming'), so the package must be installed from GitHub via devtools::install_github("dimitrif/eicCluster").
- The workflow depends on interactive expert judgement inside the Shiny app; it is not a fully automated one-shot pipeline.
- No changelog was found in the repository, making it harder to track changes to the clustering behaviour over time.
- The method is tailored to HPTLC-HRMS (planar chromatography) data; transferability to other mass spectrometry acquisition modes is not established in the provided context.

## Evidence

- [intro] Based on clusterisation of the extracted ion chromatograms, this method intends to separate the ions of interest from the background ions in high-performance thin-layer chromatography coupled with: "Based on clusterisation of the extracted ion chromatograms, this method intends to separate the ions of interest from the background ions in high-performance thin-layer chromatography coupled with"
- [readme] The only function of the package launch a shiny app where the user can interactively analyse the data.: "The only function of the package launch a shiny app where the user can interactively analyse the data."
- [readme] eicCluster::run_eicCluster(): "eicCluster::run_eicCluster()"
