---
name: biomarker-metabolite-module-mining
description: Use when you have a cleaned metabolite abundance matrix (e.g. tidyMass
  peak-area feature table) spanning many samples and need to group metabolites into
  co-accumulation modules and nominate hub metabolites as biomarker candidates when
  traditional group-based DAM comparison is impractical.
when_to_use_negative:
- The goal is a straightforward group-based differential abundance metabolite (DAM)
  comparison between a few sample groups — the iterative module approach is for population-scale
  datasets where such comparison is impractical
- Input is raw LC-MS vendor data requiring peak picking and data cleaning — upstream
  processing is handled by tidyMass before this module-mining step
- The input matrix is a .csv/.xlsx file or sample IDs contain spaces, special symbols,
  or pure numbers — the WGCNA-shinyApp only accepts tab-delimited files with clean
  sample IDs
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0605
tools:
- name: MetMiner
  role: Hosts the iterative WGCNA module-refinement workflow for metabolomics data
    via its R Shiny app
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: Standalone user-friendly shiny app for running the WGCNA module clustering
    and kME hub detection
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: MDAtoolkits
  role: Companion statistical/metabolite-analysis toolkit loaded into the MetMiner
    environment
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: tidyMass
  role: Provides the upstream cleaned peak-area feature table that serves as the abundance
    matrix input
- name: weighted gene co-expression network analysis
  role: 'Core algorithm: metabolites are networked on sample-wise abundance profiles,
    clustered into modules, and ranked by kME'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:7a911353a73b801c918c18420a674249f572b411761879e855194bf7cc0261c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - WGCNA-shinyApp
  - tidyMass
  - weighted gene co-expression network analysis
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# biomarker-metabolite-module-mining

## Summary

Iterative WGCNA-based clustering of metabolite abundance profiles (as implemented in MetMiner/WGCNA-shinyApp) that assigns every metabolite to a co-accumulation module and flags per-module hub metabolites at kME > 0.8 for biomarker screening in population-scale plant metabolomics.

## When to use

Use when you have a cleaned metabolite abundance matrix (e.g. tidyMass peak-area feature table) spanning many samples and need to group metabolites into co-accumulation modules and nominate hub metabolites as biomarker candidates when traditional group-based DAM comparison is impractical.

## When NOT to use

- The goal is a straightforward group-based differential abundance metabolite (DAM) comparison between a few sample groups — the iterative module approach is for population-scale datasets where such comparison is impractical
- Input is raw LC-MS vendor data requiring peak picking and data cleaning — upstream processing is handled by tidyMass before this module-mining step
- The input matrix is a .csv/.xlsx file or sample IDs contain spaces, special symbols, or pure numbers — the WGCNA-shinyApp only accepts tab-delimited files with clean sample IDs

## Inputs

- Cleaned metabolite abundance matrix (peak-area feature table from tidyMass or the MetMiner demo files on Google Drive/Baidu Netdisk)
- Tab-delimited expression matrix (.txt/.tsv) with metabolite IDs in rows and sample IDs in columns
- Optional trait table (qualitative or quantitative sample traits) for module–trait association

## Outputs

- Converged metabolite-to-module assignment table (CSV)
- Per-module hub-metabolite list filtered at kME > 0.8 (CSV)
- Biomarker metabolite candidates from each co-accumulation module

## How to apply

Load the cleaned metabolite abundance matrix into R together with tidyverse, tidymass, and MDAtoolkits, then launch the analysis environment via run_metMiner(maxRequestSize = 300) or the companion WGCNA-shinyApp, uploading the abundance table as a tab-delimited file with metabolite IDs in rows and sample IDs in columns (sample IDs must not contain spaces or special symbols and must not be pure numbers). Build a weighted co-expression network treating metabolites as nodes and sample-wise abundance profiles as expression vectors, and cluster metabolites into initial modules. Compute kME (module eigengene-based connectivity) for every metabolite in each module, then remove unassigned metabolites and low-kME metabolites, rebuild the network, and re-cluster — repeating this iteration until every remaining metabolite is assigned to a module. Export the converged metabolite-to-module assignment table and the per-module hub-metabolite list filtered at kME > 0.8 as CSV files.

## Related tools

- **MetMiner** (Hosts the iterative WGCNA module-refinement workflow for metabolomics data via its R Shiny app) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone user-friendly shiny app for running the WGCNA module clustering and kME hub detection) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **MDAtoolkits** (Companion statistical/metabolite-analysis toolkit loaded into the MetMiner environment) — https://github.com/ShawnWx2019/MDAtoolkits
- **tidyMass** (Provides the upstream cleaned peak-area feature table that serves as the abundance matrix input)
- **weighted gene co-expression network analysis** (Core algorithm: metabolites are networked on sample-wise abundance profiles, clustered into modules, and ranked by kME)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- The final assignment table contains no unclustered metabolites — iteration has converged
- Hub metabolites are correctly flagged as those with kME > 0.8 in their module
- Input file is tab-delimited with metabolite IDs in rows, sample IDs in columns, and no pure-number or special-symbol sample IDs
- Module structure is reproducible across re-runs of the iterative filter-and-recluster loop
- Hub-metabolite lists are exported as CSV per module, consistent with the converged assignment table

## Limitations

- The iteration repeatedly discards metabolites lacking consistent accumulation patterns across samples, so weakly varying metabolites never reach module assignment
- Linux deployment requires a graphic interface for the shiny apps
- The companion shiny apps have no stated persistence/availability guarantee for hosted instances
- MetMiner is marked as Experimental lifecycle; README notes unimplemented TODOs such as semi-annotation

## Evidence

- [abstract] iterative WGCNA until full assignment: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [intro] WGCNA as MetMiner feature (iii): "(iii) An iterative weighted gene co‐expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency."
- [discussion] shiny app availability: [evidence span withheld — anchor status: not_found]
- [readme] input format requirements: [evidence span withheld — anchor status: not_found]
- [readme] launch command: "run_metMiner(maxRequestSize = 300)"
