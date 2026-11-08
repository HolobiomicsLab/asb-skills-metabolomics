---
name: hub-metabolite-kme-screening
description: Use when a metabolite abundance matrix (e.g. tidyMass-processed peak-area
  feature table) has been clustered into WGCNA co-accumulation modules and you need
  to rank metabolites within each module and identify hub metabolites as biomarker
  candidates.
when_to_use_negative:
- Input is still raw LC-MS data needing peak picking and data cleaning — run the tidyMass-driven
  upstream processing in MetMiner first
- The dataset is small enough that standard group-based differential abundance comparison
  (e.g. pairwise DAM tests) is practical — direct group comparison is more appropriate
  than co-accumulation module mining
- 'Sample IDs contain spaces or special symbols (- @ * & #), start with numbers, or
  IDs are pure numbers — the WGCNA-shinyApp rejects such matrices'
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetMiner
  role: R Shiny pipeline hosting the iterative WGCNA strategy for metabolite module
    assignment and hub screening
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: Standalone Shiny app performing WGCNA module construction, kME computation,
    and hub identification on abundance matrices
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: MDAtoolkits
  role: Companion R package of statistical analysis tools loaded alongside MetMiner
    for downstream analysis
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: tidyMass
  role: Provides the upstream peak picking and data cleaning that produces the cleaned
    abundance matrix fed to WGCNA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:c4b6b8897372b3ae7c99deba037765cfadd1e6f3eea89703ab8cc4e72d91a235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - WGCNA-shinyApp
  - tidyMass
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hub-metabolite-kme-screening

## Summary

Screens for hub metabolites within co-accumulation modules produced by MetMiner's iterative WGCNA, using module eigengene-based connectivity (kME) with a kME > 0.8 cutoff to flag per-module hub metabolites for biomarker prioritization.

## When to use

Use when a metabolite abundance matrix (e.g. tidyMass-processed peak-area feature table) has been clustered into WGCNA co-accumulation modules and you need to rank metabolites within each module and identify hub metabolites as biomarker candidates.

## When NOT to use

- Input is still raw LC-MS data needing peak picking and data cleaning — run the tidyMass-driven upstream processing in MetMiner first
- The dataset is small enough that standard group-based differential abundance comparison (e.g. pairwise DAM tests) is practical — direct group comparison is more appropriate than co-accumulation module mining
- Sample IDs contain spaces or special symbols (- @ * & #), start with numbers, or IDs are pure numbers — the WGCNA-shinyApp rejects such matrices

## Inputs

- Cleaned metabolite abundance matrix (peak area feature table, metabolites as rows / samples as columns; tab-delimited .txt or .tsv)
- Optional trait table (sample_id plus quantitative or qualitative traits) for module-trait association
- Converged module assignment table from iterative WGCNA

## Outputs

- Metabolite-to-module assignment table (CSV)
- Per-module hub-metabolite list filtered at kME > 0.8 (CSV)
- Module eigengene connectivity (kME) values per metabolite per module

## How to apply

Build a weighted co-expression network treating metabolites as nodes and sample-wise abundance profiles as expression vectors, then cluster metabolites into modules. Compute kME (module eigengene-based connectivity) for every metabolite in each module; metabolites with low kME or no module assignment are removed and the network is rebuilt and re-clustered, repeating this iterative WGCNA loop until every remaining metabolite is assigned to a module. Once converged, flag hub metabolites as those with kME > 0.8 per module and export the metabolite-to-module assignment table plus the per-module hub list as CSV files. In the WGCNA-shinyApp, the input must be a tab-delimited expression matrix (.txt/.tsv) with metabolite IDs in rows and sample IDs in columns, optionally paired with a trait table for module–trait association.

## Related tools

- **MetMiner** (R Shiny pipeline hosting the iterative WGCNA strategy for metabolite module assignment and hub screening) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone Shiny app performing WGCNA module construction, kME computation, and hub identification on abundance matrices) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **MDAtoolkits** (Companion R package of statistical analysis tools loaded alongside MetMiner for downstream analysis) — https://github.com/ShawnWx2019/MDAtoolkits
- **tidyMass** (Provides the upstream peak picking and data cleaning that produces the cleaned abundance matrix fed to WGCNA)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- The iteration has converged when the final assignment table contains no unclustered metabolites (every remaining metabolite is assigned to a module)
- Hub metabolites are correctly flagged as those with kME > 0.8 within each module
- Input matrix format checks pass: tab-delimited file, metabolite IDs not pure numbers, sample IDs free of spaces and special symbols (- @ * & #) and not starting with digits
- Outputs include both a complete metabolite-to-module mapping CSV and a per-module hub list CSV
- Module assignments remain stable upon re-clustering after removal of low-kME metabolites (iterative refinement converged rather than oscillating)

## Limitations

- MetMiner is lifecycle-tagged Experimental and no version numbers are stated for MetMiner, tidyMass, or MDAtoolkits, hindering exact reproducibility
- No containerization (Docker/Singularity) is documented for reproducible deployment of the Shiny apps
- No persistence/availability guarantee is given for the hosted shiny apps; the standalone WGCNA-shinyApp must be cloned and run locally (Linux requires a graphic interface)
- The iterative refiltering loop may require multiple rebuild-and-recluster cycles on very large population-scale matrices; convergence time is not bounded in the documentation

## Evidence

- [abstract] iterative WGCNA hub screening: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [intro] kME cutoff: "(iii) An iterative weighted gene co‐expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency."
- [results] iterative refiltering: [evidence span withheld — anchor status: not_found]
- [readme] input format requirements: [evidence span withheld — anchor status: external_doc]
- [readme] noise filtering rationale: [evidence span withheld — anchor status: external_doc]
