---
name: shiny-app-driven-metabolomics-pipeline-execution
description: Use when the goal is to run plant LC–MS metabolomics analysis (data cleaning,
  annotation against the PM database, statistics, publication-ready figures) interactively
  via a GUI, especially for users without programming experience or on large-scale
  datasets.
when_to_use_negative:
- The analysis must run headless or scripted in CI — the Shiny app requires an interactive
  session (on Linux it must have a graphic interface), so use scripted tidyMass/tidyverse
  pipelines instead
- The metabolomics data are non-plant — the integrated PM database is plant-specific,
  so annotation accuracy depends on plant metabolites being the target
- The task is gene–metabolite co-expression mining via WGCNA — use the separate WGCNA-shinyApp
  rather than the MetMiner annotation/filtering panels
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMiner
  role: 'Core Shiny application that hosts the full workflow: parameter setting, data
    cleaning, compound annotation, annotation filtering, and statistics'
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Drives upstream data processing (peak picking and data cleaning) inside the
    pipeline
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- name: MDAtoolkits
  role: Provides statistical analysis, metabolite classification, and enrichment analysis
    tools used downstream in the app
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: R shiny
  role: Deployment framework allowing the app to run on servers with additional computational
    resources for large-scale datasets
- name: PM database (plant metabolomics database)
  role: Plant-specific metabolite database integrated into MetMiner to enhance accuracy
    and speed of metabolite annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:913643aa9b4b487514588dbaae698094b2a6a870dfbe14e08d36f96de19a800b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - PM database (plant metabolomics database)
  - R shiny
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# shiny-app-driven-metabolomics-pipeline-execution

## Summary

Executes the MetMiner R Shiny application to run the full plant metabolomics workflow — peak picking/cleaning via tidyMass, metabolite annotation against the integrated plant-specific (PM) database, and downstream statistics — through a graphical interface rather than scripted code.

## When to use

Use when the goal is to run plant LC–MS metabolomics analysis (data cleaning, annotation against the PM database, statistics, publication-ready figures) interactively via a GUI, especially for users without programming experience or on large-scale datasets.

## When NOT to use

- The analysis must run headless or scripted in CI — the Shiny app requires an interactive session (on Linux it must have a graphic interface), so use scripted tidyMass/tidyverse pipelines instead
- The metabolomics data are non-plant — the integrated PM database is plant-specific, so annotation accuracy depends on plant metabolites being the target
- The task is gene–metabolite co-expression mining via WGCNA — use the separate WGCNA-shinyApp rather than the MetMiner annotation/filtering panels

## Inputs

- Matched LC–MS feature tables (e.g., Thermo and Waters platform tables, 1,901 near-identical features) in tab-delimited format
- MetMiner demo files (Google Drive or Baidu Netdisk)
- Raw metabolomics datasets for tidyMass-driven peak picking and data cleaning
- Integrated plant-specific metabolite (PM) database with MS1 and MS2 spectra

## Outputs

- Per-dataset annotated metabolite lists (e.g., 536 annotations for Thermo, 447 for Waters)
- Filtered annotation tables produced by filter_annotations_massdataset
- Cross-platform overlap counts (e.g., 257 shared metabolites; 48%/58% of Thermo/Waters annotations)
- Publication-ready figures downloadable as pdf, tiff, png, or jpg

## How to apply

Install MetMiner and its dependency stack from GitHub with remotes::install_github('ShawnWx2019/MetMiner') after installing tidymass (via install_tidymass), MDAtoolkits, ropls, PCAtools, ComplexHeatmap, and the shiny UI packages. In R, load tidyverse, tidymass, MDAtoolkits, and MetMiner, then launch the app with run_metMiner(maxRequestSize = 300), where 300 sets the maximum upload size in MB for large feature tables. Load matched feature tables (e.g., the demo Thermo and Waters platform tables with 1,901 near-identical features) through the Shiny file interface, run the compound annotation module against the integrated plant-specific PM database (retrieving records via get_compound_info per feature), then apply filter_annotations_massdataset(object, annotate_tbl, method) to retain reliable annotations. The rationale for the Shiny architecture is that MetMiner can be deployed on servers to use extra compute for large datasets while interactive visualization supports most operations during data cleaning and metabolite annotation; annotation/filtering parameters should be revisited whenever results deviate from validated case-study values.

## Related tools

- **MetMiner** (Core Shiny application that hosts the full workflow: parameter setting, data cleaning, compound annotation, annotation filtering, and statistics) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Drives upstream data processing (peak picking and data cleaning) inside the pipeline) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- **MDAtoolkits** (Provides statistical analysis, metabolite classification, and enrichment analysis tools used downstream in the app) — https://github.com/ShawnWx2019/MDAtoolkits
- **R shiny** (Deployment framework allowing the app to run on servers with additional computational resources for large-scale datasets)
- **PM database (plant metabolomics database)** (Plant-specific metabolite database integrated into MetMiner to enhance accuracy and speed of metabolite annotation)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Reproduce the case-study annotation counts exactly: 536 metabolites annotated in the Thermo dataset, 447 in Waters, 257 shared — any deviation flags annotation or filtering parameters for expert review
- Verify computed cross-platform overlap equals 48% of Thermo and 58% of Waters annotations
- Confirm the app launches successfully with run_metMiner(maxRequestSize = 300) after loading tidyverse, tidymass, MDAtoolkits, and MetMiner
- Check that filter_annotations_massdataset(object, annotate_tbl, method) returns only annotations deemed reliable under the chosen method
- Confirm figure download buttons export pdf, tiff, png, and jpg formats (added in MetMiner v1.0.1)

## Limitations

- In-source fragmentation products can be misannotated as genuine metabolites; no satisfactory detection/removal solution exists yet and fixing this is a focus of the next MetMiner version
- No version numbers, release DOI, containerization (Docker/Singularity), or persistence guarantee for hosted shiny apps are stated; the GitHub lifecycle badge is 'Experimental'
- Large-scale feature tables must fit the maxRequestSize limit (300 MB in the documented invocation), and demo files are hosted on Google Drive/Baidu Netdisk rather than a versioned release

## Evidence

- [abstract] Built on R shiny, MetMiner can be deployed on servers to utilize additional computational resources for processing large-scale datasets: "Built on R shiny, MetMiner can be deployed on servers to utilize additional computational resources for processing large-scale datasets"
- [intro] MetMiner enables users without programming experience to easily navigate the whole analytical process: "MetMiner enables users without programming experience to easily navigate the whole analytical process based on the experimental design and to generate publication‐ready figures"
- [discussion] interactive visualization is provided in most operations during data cleaning and metabolite annotation: "interactive visualization is provided in most operations during data cleaning and metabolite annotation"
- [readme] run_metMiner(maxRequestSize = 300) launches the Shiny app: "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
- [methods] filter_annotations_massdataset(object, annotate_tbl, method): "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
