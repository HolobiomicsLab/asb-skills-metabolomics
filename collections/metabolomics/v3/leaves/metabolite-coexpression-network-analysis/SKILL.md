---
name: metabolite-coexpression-network-analysis
description: Use when a metabolomics abundance matrix (e.g., peak areas from pseudotargeted
  LC-MS across many samples/mutants) must be partitioned into co-accumulation modules,
  or when group-based DAM comparison is impractical for population-scale data.
when_to_use_negative:
- When the goal is pairwise group-based differential abundance metabolite (DAM) comparison
  between defined sample groups — the article states this strategy is for when traditional
  group-based DAM comparison is impractical.
- When the input is raw vendor LC-MS data rather than a cleaned/normalized peak-area
  table — upstream peak picking and data cleaning must first be done with tidyMass.
- When feature IDs are pure numbers or the table is .csv/.xlsx — the WGCNA-shinyApp
  explicitly rejects these input formats.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: MetMiner
  role: R Shiny pipeline hosting the iterative WGCNA data-mining workflow for metabolomics
    abundance tables
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: Standalone user-friendly Shiny app that runs the weighted gene co-expression
    network analysis (network construction, module detection, kME, hub selection)
    on peak-area matrices
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: MDAtoolkits
  role: Downstream classification/enrichment analysis of assigned modules (e.g., ClassyFire
    term over-representation)
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: tidyMass
  role: Upstream peak picking and data cleaning that produces the WGCNA-compatible
    abundance matrix
- name: weighted gene co-expression network analysis
  role: 'Core statistical method: pairwise metabolite correlations, soft-threshold
    power selection, dynamic tree cutting, module eigengenes and kME'
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:68029daa6b9320a3cd39ae1b83c15bbd601c3c10696ec4aa10baa2c2cce5f90d
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-coexpression-network-analysis

## Summary

Clusters metabolites into co-accumulation modules using an iterative weighted gene co-expression network analysis (WGCNA) strategy implemented in MetMiner/WGCNA-shinyApp, treating metabolites as nodes and sample-wise peak-area profiles as expression vectors, to screen biomarker metabolites and assign module membership when group-based differential abundance comparison is impractical.

## When to use

Use when a metabolomics abundance matrix (e.g., peak areas from pseudotargeted LC-MS across many samples/mutants) must be partitioned into co-accumulation modules, or when group-based DAM comparison is impractical for population-scale data.

## When NOT to use

- When the goal is pairwise group-based differential abundance metabolite (DAM) comparison between defined sample groups — the article states this strategy is for when traditional group-based DAM comparison is impractical.
- When the input is raw vendor LC-MS data rather than a cleaned/normalized peak-area table — upstream peak picking and data cleaning must first be done with tidyMass.
- When feature IDs are pure numbers or the table is .csv/.xlsx — the WGCNA-shinyApp explicitly rejects these input formats.

## Inputs

- Tab-delimited (.txt/.tsv) abundance/expression matrix: metabolite IDs in rows, sample IDs in columns; for metabolomics use LC-MS peak areas (e.g., corrected peak-picking table of 1,962 metabolites across 206 Arabidopsis F-box mutant samples)
- Optional trait table (.txt) with sample_id column plus qualitative (treat/control) or quantitative traits
- tidyMass-processed feature table or MetMiner demo files (Google Drive/Baidu Netdisk)

## Outputs

- Metabolite-to-module assignment table (CSV) with no unclustered metabolites at convergence
- Per-module hub-metabolite list filtered at kME > 0.8 (CSV)
- Module eigengenes and module membership (kME) values per metabolite
- Optionally, per-module ClassyFire classification over-representation enrichment results

## How to apply

Load the cleaned metabolite peak-area matrix (from a tidyMass-processed feature table or the MetMiner demo files) with rows as metabolite IDs and columns as sample IDs — only tab-delimited .txt/.tsv is accepted, sample IDs must not contain spaces or special symbols like `- @ * & #` and must not start with numbers, and pure-numeric metabolite IDs are forbidden. Select the `peak area` format and choose a normalization method (`raw` or log10(x+1)); optionally filter low-abundance features by a sample-percentage/expression cutoff and select top-variable features by MAD or variance. Build the weighted co-expression network, choose the soft-thresholding power from the scale-free topology fit, and detect modules with dynamic tree cutting; then compute kME (module eigengene-based connectivity) for every metabolite, remove unassigned and low-kME metabolites, rebuild the network, and repeat these WGCNA rounds until all remaining metabolites are assigned to modules (in the Arabidopsis 206 F-box mutant case study, 1,365 of 1,962 metabolites converged into 12 modules after four rounds). Export the converged metabolite-to-module assignment table plus per-module hub metabolites filtered at kME > 0.8, then optionally map ClassyFire classifications onto the modules and run over-representation enrichment with MDAtoolkits.

## Related tools

- **MetMiner** (R Shiny pipeline hosting the iterative WGCNA data-mining workflow for metabolomics abundance tables) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone user-friendly Shiny app that runs the weighted gene co-expression network analysis (network construction, module detection, kME, hub selection) on peak-area matrices) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **MDAtoolkits** (Downstream classification/enrichment analysis of assigned modules (e.g., ClassyFire term over-representation)) — https://github.com/ShawnWx2019/MDAtoolkits
- **tidyMass** (Upstream peak picking and data cleaning that produces the WGCNA-compatible abundance matrix)
- **weighted gene co-expression network analysis** (Core statistical method: pairwise metabolite correlations, soft-threshold power selection, dynamic tree cutting, module eigengenes and kME)

## Examples

```
library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Convergence check: the final assignment table contains no unassigned metabolites (iteration converged)
- Case-study invariant: iterative WGCNA on 1,962 metabolites from the 206 Arabidopsis F-box mutant dataset yields 1,365 metabolites in 12 modules after four rounds
- Enrichment sanity: seven of the modules show significant ClassyFire term enrichment; otherwise flag the run as diverging from the reported outcome
- Hub definition check: hub metabolites are exactly those with kME > 0.8 per module
- Input format validation: file is tab-delimited, metabolite IDs are non-numeric, and sample IDs contain no spaces or special symbols

## Limitations

- In-source fragmentation products can be misannotated as genuine metabolites, and no satisfactory solution exists to detect and remove such redundant annotations (a focus for the next MetMiner version)
- For RNAseq-like count inputs, variance-stabilizing transformation (DESeq2 vst) or log10(x+1) is required before WGCNA; low-expression features that are mostly zero produce meaningless correlations and should be removed
- The choice of filtering thresholds (sample percentage, expression cutoff, MAD/VAR top-N) depends on experimental design and data depth, and filtering by mean vs. variance is a matter of debate
- Linux requires a graphic interface to run the Shiny app; software lifecycle is marked Experimental

## Evidence

- [results] Iterative WGCNA on the peak areas of 1,962 metabolites from the 206 F-box mutant dataset grouped 1,365 metabolites into 12 modules after four rounds of WGCNA.: [evidence span withheld — anchor status: not_found]
- [abstract] we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [discussion] Given the complexity of WGCNA analysis, a separate user-friendly WGCNA-shinyapp (https://github.com/ShawnWx2019/WGCNA-shinyApp) is developed.: [evidence span withheld — anchor status: not_found]
- [readme] Only accepted tab-delimited file, such as `.txt` or `.tsv`, DO NOT use `.csv`, `.xlsx`, `.xls`.: [evidence span withheld — anchor status: external_doc]
- [other] Export the converged module assignment table and the per-module hub-metabolite list filtered at kME > 0.8 as CSV files.: [evidence span withheld — anchor status: not_found]
