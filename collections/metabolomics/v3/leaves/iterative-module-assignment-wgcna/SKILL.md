---
name: iterative-module-assignment-wgcna
description: Use when a large metabolite abundance matrix (peak-area feature table,
  metabolites × samples) must be partitioned into co-accumulation modules and hub
  metabolites, and group-based DAM comparison is impractical.
when_to_use_negative:
- When the analysis goal is differential abundance testing between defined groups
  rather than unsupervised co-accumulation module discovery — the paper frames this
  as the alternative when 'traditional group-based DAM comparison is impractical'
- When the abundance table is not cleaned/normalized upstream (e.g. raw unfiltered
  peak areas) — the app expects pre-cleaned 'peak area' values with noise filtering
  applied beforehand
- When data are .csv/.xlsx/.xls — the WGCNA-shinyApp only accepts tab-delimited files
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMiner
  role: Hosts the iterative WGCNA biomarker-screening strategy inside its shiny pipeline
    for large-scale plant metabolomics data
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: Standalone click-driven app performing expression-matrix filtering, module
    clustering, and kME-based hub detection
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: MDAtoolkits
  role: R companion package loaded into the MetMiner analysis environment for statistical
    and data-mining support
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Produces the upstream cleaned/normalized feature table consumed as the WGCNA
    expression matrix
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:40c9495f0530b0f0331988b1b049dbb958c066b7efee34c9e020f3052cf40728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - WGCNA-shinyApp
  - tidyMass
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Iterative module assignment WGCNA

## Summary

An iterative weighted gene co-expression network analysis (WGCNA) strategy, implemented in MetMiner and the companion WGCNA-shinyApp, that repeatedly removes unclustered and low-kME metabolites and rebuilds the network until every metabolite is assigned to a co-accumulation module, enabling biomarker metabolite screening in population-scale plant metabolomics data.

## When to use

Use when a large metabolite abundance matrix (peak-area feature table, metabolites × samples) must be partitioned into co-accumulation modules and hub metabolites, and group-based DAM comparison is impractical.

## When NOT to use

- When the analysis goal is differential abundance testing between defined groups rather than unsupervised co-accumulation module discovery — the paper frames this as the alternative when 'traditional group-based DAM comparison is impractical'
- When the abundance table is not cleaned/normalized upstream (e.g. raw unfiltered peak areas) — the app expects pre-cleaned 'peak area' values with noise filtering applied beforehand
- When data are .csv/.xlsx/.xls — the WGCNA-shinyApp only accepts tab-delimited files

## Inputs

- Tab-delimited metabolite abundance matrix (.txt/.tsv; metabolite ID per row, sample ID per column; peak area values from LC-MS)
- Optional trait table (.txt/.tsv) with sample_id column plus quantitative or qualitative traits (format1 two-column traits auto-converted to 0/1 format2)
- MetMiner demo abundance files from Google Drive or Baidu Netdisk for testing

## Outputs

- Metabolite-to-module assignment table (CSV) with no unclustered metabolites
- Per-module hub-metabolite list filtered at kME > 0.8 (CSV)
- Module eigengene/trait association outputs from the shiny app

## How to apply

Load the cleaned metabolite abundance matrix (e.g. a tidyMass-processed feature table or the MetMiner demo files from Google Drive/Baidu Netdisk) into R with tidyverse, tidymass, and MDAtoolkits, then launch run_metMiner(maxRequestSize = 300) or the WGCNA-shinyApp (started via Rscript WGCNAbyClick.v1.R). Input must be a tab-delimited .txt/.tsv matrix with metabolite IDs in rows and sample IDs in columns; sample IDs must not contain spaces or special symbols and IDs must not be pure numbers. Build a weighted network treating metabolites as nodes and sample-wise abundance profiles as expression vectors, cluster into initial modules, compute kME (module eigengene-based connectivity) for each metabolite, then remove unassigned and low-kME metabolites and rebuild/re-cluster. Repeat the filter–rebuild loop until no metabolite remains unclustered. Export the converged metabolite-to-module assignment table and per-module hub lists filtered at kME > 0.8. Prior to clustering, filter noisy features (e.g. features below cutoff in most samples) and select top-N variable features by MAD or variance, optionally after log10(x+1) transformation of peak areas.

## Related tools

- **MetMiner** (Hosts the iterative WGCNA biomarker-screening strategy inside its shiny pipeline for large-scale plant metabolomics data) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone click-driven app performing expression-matrix filtering, module clustering, and kME-based hub detection) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **MDAtoolkits** (R companion package loaded into the MetMiner analysis environment for statistical and data-mining support) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Produces the upstream cleaned/normalized feature table consumed as the WGCNA expression matrix) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt

## Examples

```
Rscript WGCNAbyClick.v1.R
```

## Evaluation signals

- Convergence check: the final module assignment table contains no unassigned (unclustered) metabolites
- Hub correctness: per-module hub lists are flagged exactly at kME > 0.8
- Input-format check: matrix is tab-delimited with sample IDs free of spaces/special symbols, no pure-numeric IDs, and metabolite IDs in rows
- Pre-filter effectiveness: features consistently low across samples (noise) and top-N MAD/VAR-selected features are removed before network construction
- Reproducibility: the same iteration yields the same converged assignment when re-run from the cleaned tidyMass feature table

## Limitations

- The app lifecycle is 'Experimental' and, per the article's own missing-signal notes, no containerization (Docker/Singularity), version pinning, or persistence guarantee for the hosted shiny apps is provided
- Sample IDs containing spaces or special symbols (- @ * & #), or pure-numeric metabolite IDs, break the app's input expectations
- Only tab-delimited .txt/.tsv accepted; .csv/.xlsx/.xls must be converted first
