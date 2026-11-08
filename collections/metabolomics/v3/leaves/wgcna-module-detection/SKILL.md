---
name: wgcna-module-detection
description: Use when you have a metabolite-by-sample quantification matrix (e.g.
  LC-MS peak areas from a pseudotargeted metabolomics run) and need to group co-accumulating
  metabolites into modules for biomarker metabolite screening.
when_to_use_negative:
- Input is still raw vendor LC-MS data requiring peak picking and cleaning — run the
  tidyMass-driven upstream processing in MetMiner first to obtain the peak area table
- The expression file is .csv, .xlsx, or .xls, or contains sample IDs with spaces/special
  symbols or purely numeric metabolite IDs — the WGCNA-shinyApp only accepts tab-delimited
  files and rejects these formats
- The goal is annotation, metabolite identification, or statistical testing rather
  than co-accumulation module detection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetMiner
  role: R Shiny pipeline that loads the tidymass-formatted peak table and drives the
    iterative WGCNA biomarker-screening strategy end to end
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: 'Standalone Shiny app dedicated to WGCNA module detection: expression-matrix
    filtering (noise removal, MAD/VAR top-N), soft-thresholding, and dynamic tree
    cut module assignment on peak-area inputs'
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: MDAtoolkits
  role: 'Post-module-detection step: maps ClassyFire compound classifications onto
    module members and performs over-representation enrichment per module'
  repo: https://github.com/ShawnWx2019/MDAtoolkits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:42f84e9ea313649aec0ee65526518be07611f0b573285f1fb9f8995c67d44488
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - WGCNA-shinyApp
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# wgcna-module-detection

## Summary

Detects co-accumulation modules among metabolites using MetMiner's iterative weighted gene co-expression network analysis (WGCNA) strategy, in which the network is rebuilt after removing the weakest module member each round to sharpen biomarker metabolite screening.

## When to use

Use when you have a metabolite-by-sample quantification matrix (e.g. LC-MS peak areas from a pseudotargeted metabolomics run) and need to group co-accumulating metabolites into modules for biomarker metabolite screening.

## When NOT to use

- Input is still raw vendor LC-MS data requiring peak picking and cleaning — run the tidyMass-driven upstream processing in MetMiner first to obtain the peak area table
- The expression file is .csv, .xlsx, or .xls, or contains sample IDs with spaces/special symbols or purely numeric metabolite IDs — the WGCNA-shinyApp only accepts tab-delimited files and rejects these formats
- The goal is annotation, metabolite identification, or statistical testing rather than co-accumulation module detection

## Inputs

- Tab-delimited expression matrix (.txt/.tsv) of metabolite peak areas with metabolite IDs in rows and sample IDs in columns (e.g. 1,962 metabolites x 206 F-box mutant samples)
- Optional trait table mapping sample_id to quantitative or qualitative traits (e.g. treat/control)

## Outputs

- Module assignments grouping metabolites into co-accumulation modules (e.g. 12 modules covering 1,365 metabolites after four rounds)
- Per-metabolite module membership / eigengene association values used to drive the iterative removal
- Module eigengenes and module-trait relationships for biomarker screening

## How to apply

Load the peak-picking table (e.g. peak areas of 1,962 metabolite features across 206 Arabidopsis F-box mutant samples) as a WGCNA-compatible expression matrix via tidyMass/MetMiner, with metabolite IDs in rows and sample IDs in columns saved as a tab-delimited .txt/.tsv. Clean the matrix first per the WGCNA FAQ guidance implemented in WGCNA-shinyApp: remove features whose values fall below a cutoff in a chosen percentage of samples, then filter the top-N features by median absolute deviation (MAD) or variance (VAR). Run WGCNA by computing pairwise metabolite correlations, choosing the soft-thresholding power from the scale-free topology fit, and cutting modules with dynamic tree cutting. Then execute the iterative refinement: identify the metabolite with the lowest module membership (highest module eigengene dissimilarity), remove it, and rebuild the network, repeating over four rounds. In the reference case study this grouped 1,365 metabolites into 12 modules after four rounds; the final module assignments can then be passed to MDAtoolkits for ClassyFire classification and per-module over-representation enrichment.

## Related tools

- **MetMiner** (R Shiny pipeline that loads the tidymass-formatted peak table and drives the iterative WGCNA biomarker-screening strategy end to end) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone Shiny app dedicated to WGCNA module detection: expression-matrix filtering (noise removal, MAD/VAR top-N), soft-thresholding, and dynamic tree cut module assignment on peak-area inputs) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **MDAtoolkits** (Post-module-detection step: maps ClassyFire compound classifications onto module members and performs over-representation enrichment per module) — https://github.com/ShawnWx2019/MDAtoolkits

## Examples

```
git clone git@github.com:ShawnWx2019/WGCNA-shinyApp.git WGCNAshiny; cd WGCNAshiny; Rscript WGCNAbyClick.v1.R
```

## Evaluation signals

- Reproducing the reference run: four rounds of iterative WGCNA on the 206 F-box mutant peak-area dataset yield exactly 12 modules covering all 1,365 metabolites
- Seven of the resulting modules show significant ClassyFire term enrichment via MDAtoolkits classification/enrichment tools
- Input passes format validation: tab-delimited file, metabolite IDs in rows, sample IDs in columns, no pure-numeric IDs and no spaces/special symbols in sample IDs
- The soft-thresholding power was selected from a scale-free topology fit, and modules were cut with dynamic tree cutting rather than fixed-height clustering
- At each iteration, the removed metabolite is the one with the lowest module membership (highest module eigengene dissimilarity), and the network is rebuilt rather than only re-cut

## Limitations

- Suspected in-source fragmentation products can be misannotated as genuine metabolites and the authors note no satisfactory solution yet exists to detect and remove these redundant annotations before network construction
- The tools are marked Experimental lifecycle, MIT-licensed, with no stated version numbers, no containerization, and no persistence guarantee for the hosted Shiny apps
- Low-abundance/noisy features degrade correlation-based module detection, so the pre-filtering (cutoff-in-percentage-of-samples plus MAD/VAR top-N) thresholds must be tuned to experimental design and sample count

## Evidence

- [abstract] iterative WGCNA strategy: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [results] case-study result: [evidence span withheld — anchor status: not_found]
- [readme] input format: [evidence span withheld — anchor status: external_doc]
- [readme] peak-area input and feature filtering: [evidence span withheld — anchor status: external_doc]
- [readme] MAD/VAR filtering step: [evidence span withheld — anchor status: external_doc]
- [intro] rationale for iterative strategy: "An iterative weighted gene co‐expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency."
