---
name: metabolite-module-enrichment-analysis
description: Use when a metabolite co-expression/co-accumulation network (e.g. WGCNA
  modules built from LC-MS peak areas) must be interpreted biologically by testing
  whether specific ClassyFire compound classes are over-represented within each module.
when_to_use_negative:
- Input is still raw LC-MS data or an uncleaned peak table — upstream peak picking
  and data cleaning via tidyMass must be completed before modules exist to enrich
- Only a small set of metabolites without ClassyFire annotations is available, so
  per-module over-representation of compound classes is not statistically meaningful
- The goal is gene (transcriptomics) module enrichment rather than metabolite compound-class
  enrichment
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0203
tools:
- name: MDAtoolkits
  role: Provides the metabolite classification and enrichment analysis functions used
    to map ClassyFire terms and test per-module over-representation
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: MetMiner
  role: R Shiny pipeline hosting the iterative WGCNA step that produces the metabolite
    modules being enriched, and the interface through which downstream classification/enrichment
    is run
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: Standalone user-friendly app for the WGCNA module construction whose output
    (module assignments) feeds the enrichment step
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: tidymass
  role: Drives upstream peak picking and data cleaning so that a reproducible, cleaned
    peak-area matrix exists before module construction and enrichment
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:19c2e7ca42654ac0883554f1b3fdc37758975bc38d354b3b113828204747efaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MDAtoolkits
  tools:
  - MDAtoolkits
  - MetMiner
  - WGCNA-shinyApp
  - tidymass
  - weighted gene co-expression network analysis
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolite-module-enrichment-analysis

## Summary

Annotates metabolite co-accumulation modules produced by MetMiner's iterative WGCNA with ClassyFire compound classifications and runs per-module over-representation (enrichment) analysis using the MDAtoolkits classification/enrichment tools, giving biological meaning to co-abundance network modules in plant metabolomics data.

## When to use

Use when a metabolite co-expression/co-accumulation network (e.g. WGCNA modules built from LC-MS peak areas) must be interpreted biologically by testing whether specific ClassyFire compound classes are over-represented within each module.

## When NOT to use

- Input is still raw LC-MS data or an uncleaned peak table — upstream peak picking and data cleaning via tidyMass must be completed before modules exist to enrich
- Only a small set of metabolites without ClassyFire annotations is available, so per-module over-representation of compound classes is not statistically meaningful
- The goal is gene (transcriptomics) module enrichment rather than metabolite compound-class enrichment

## Inputs

- Final WGCNA module assignment table (metabolite feature IDs mapped to module colors/labels) exported after the last WGCNA round
- Peak-area expression matrix (tab-delimited .txt/.tsv, metabolite IDs in rows, sample IDs in columns) used to build the modules
- ClassyFire compound classification table for the annotated metabolite features

## Outputs

- Per-module over-representation/enrichment results for ClassyFire compound-class terms
- Class-annotated module composition table
- Publication-ready figures of module composition and enrichment generated through the MetMiner/MDAtoolkits R Shiny interfaces

## How to apply

First complete iterative WGCNA in MetMiner (e.g. four rounds on peak areas of 1,962 metabolites from the 206 Arabidopsis F-box mutant pseudotargeted dataset, yielding 1,365 metabolites in 12 modules) and export the final module assignments. Install and load MDAtoolkits via remotes from the ShawnWx2019/MDAtoolkits 'master' branch, then map ClassyFire compound classifications onto the metabolites assigned to each module. Run the MDAtoolkits classification and enrichment tools per module to test over-representation of ClassyFire terms against the classified metabolite background. Judge success by whether a defined subset of modules shows significant ClassyFire term enrichment — in the paper's case study, seven of the twelve modules were significantly enriched — and flag any run diverging from this pattern as needing review.

## Related tools

- **MDAtoolkits** (Provides the metabolite classification and enrichment analysis functions used to map ClassyFire terms and test per-module over-representation) — https://github.com/ShawnWx2019/MDAtoolkits
- **MetMiner** (R Shiny pipeline hosting the iterative WGCNA step that produces the metabolite modules being enriched, and the interface through which downstream classification/enrichment is run) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone user-friendly app for the WGCNA module construction whose output (module assignments) feeds the enrichment step) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **tidymass** (Drives upstream peak picking and data cleaning so that a reproducible, cleaned peak-area matrix exists before module construction and enrichment) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt

## Examples

```
if (!require('MDAtoolkits')) remotes::install_github('ShawnWx2019/MDAtoolkits',ref = 'master')
```

## Evaluation signals

- The number of metabolites partitioned into modules matches the WGCNA export (case study invariant: 1,365 metabolites in 12 modules after four rounds)
- A specific subset of modules shows significant ClassyFire term over-representation (case study: seven of twelve modules); runs diverging from this pattern are flagged
- Every metabolite included in enrichment has a mapped ClassyFire classification term before the test is run
- MDAtoolkits is installed from the ShawnWx2019/MDAtoolkits repo with ref='master', matching the documented installation command
- Enrichment output tables cover one result per module (12 results for the case-study run), with significance statistics per ClassyFire term

## Limitations

- Suspected in-source fragmentation products may be wrongly annotated as genuine metabolites, and no satisfactory solution yet exists to detect and remove such redundant annotations, which can contaminate module composition before enrichment
- No version number, release DOI, or availability guarantee is stated for MetMiner or MDAtoolkits, so enrichment results may not be reproducible across tool updates
- MetMiner/MDAtoolkits run as R Shiny apps requiring server or local deployment with additional computational resources for large-scale datasets

## Evidence

- [abstract] MDAtoolkits role and scope: "We have also developed MDAtoolkits, which include a complete set of tools for statistical analysis, metabolite classification, and enrichment analysis"
- [intro] iterative WGCNA origin of modules: "(iii) An iterative weighted gene co‐expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency."
- [discussion] biological validation via pseudotargeted dataset: [evidence span withheld — anchor status: not_found]
- [readme] input format requirement for module data: [evidence span withheld — anchor status: not_found]
