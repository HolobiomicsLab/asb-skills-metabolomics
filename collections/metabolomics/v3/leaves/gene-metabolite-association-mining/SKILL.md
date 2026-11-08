---
name: gene-metabolite-association-mining
description: Use when a pseudotargeted metabolomics dataset over a mutant/genotype
  panel needs to be mined for metabolite-mutation associations, e.g. ranking mutants
  by module eigenmetabolite to find genes controlling a specific metabolite like glucoraphanin.
when_to_use_negative:
- Input is raw LC-MS vendor data requiring peak picking — upstream processing is handled
  first by tidyMass within MetMiner before association mining
- The dataset is not from a genotype/mutant panel or lacks sample metadata linking
  mutants to abundance columns, so eigenmetabolite ranking across mutants is not meaningful
- The goal is metabolite annotation itself rather than gene-metabolite association
  discovery
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetMiner
  role: Shiny pipeline used to load the pseudotargeted dataset, run iterative WGCNA,
    rank mutants by eigenmetabolite, and render metabolite profile figures
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: drives upstream peak picking and data cleaning of the metabolomics data before
    association mining
- name: MDAtoolkits
  role: dependency providing statistical analysis, metabolite classification, and
    enrichment analysis supporting the mining workflow
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: weighted gene co-expression network analysis
  role: iterative WGCNA strategy that identifies metabolite modules and eigenmetabolites
    used to rank mutants
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: R shiny
  role: framework on which the MetMiner app is built for server deployment and interactive
    figure export
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:765809745d1fbb58efcd3a20dd620604f06663cc0bf8092f60ae0dd89d8a619c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - R shiny
  - tidyMass
  - weighted gene co-expression network analysis
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# gene-metabolite-association-mining

## Summary

Mining of pseudotargeted plant metabolomics datasets with MetMiner's iterative WGCNA strategy to identify gene-metabolite associations, such as F-box mutants affecting glucosinolate hub metabolites, and validate them by mutant/complementation.

## When to use

Use when a pseudotargeted metabolomics dataset over a mutant/genotype panel needs to be mined for metabolite-mutation associations, e.g. ranking mutants by module eigenmetabolite to find genes controlling a specific metabolite like glucoraphanin.

## When NOT to use

- Input is raw LC-MS vendor data requiring peak picking — upstream processing is handled first by tidyMass within MetMiner before association mining
- The dataset is not from a genotype/mutant panel or lacks sample metadata linking mutants to abundance columns, so eigenmetabolite ranking across mutants is not meaningful
- The goal is metabolite annotation itself rather than gene-metabolite association discovery

## Inputs

- pseudotargeted metabolomics abundance table (peak-area features per Arabidopsis mutant sample, e.g. MT_205/max2)
- sample metadata mapping mutant identifiers to metabolite abundance columns
- tab-delimited expression matrix (.txt/.tsv, feature IDs in rows, sample IDs in columns) as required by WGCNA-shinyApp

## Outputs

- module assignment of metabolites (e.g. green module containing hub glucosinolate metabolites)
- mutant_eigenmetabolite_ranking.csv with top-20 and bottom-20 mutants ranked on module eigenmetabolite values
- target metabolite accumulation profile figure (pdf/tiff/png/jpg via MetMiner download button)
- validated gene-metabolite associations (e.g. negative MAX2-glucoraphanin association)

## How to apply

Install MetMiner, tidyMass, and MDAtoolkits from GitHub (with ropls and PCAtools as dependencies), then launch the MetMiner Shiny app via run_metMiner(maxRequestSize = 300) to handle large-scale files. Load the pseudotargeted metabolomics dataset (metabolite abundance columns keyed to mutant identifiers such as MT_205/max2) and run the iterative weighted gene co-expression network analysis (WGCNA) module to isolate modules containing hub metabolites (e.g. the green module containing glucoraphanin). Compute module eigenmetabolite values per sample, rank mutants on this metric, and export the top-20 and bottom-20 mutants (e.g. to mutant_eigenmetabolite_ranking.csv). Extract the target metabolite's abundance across all mutant lines and render its accumulation profile (exportable as pdf/tiff/png/jpg via the MetMiner download button). Validate that candidate mutants (e.g. max2) rank among the extremes for the module eigenmetabolite and show the expected metabolite phenotype; a complementary expression restoring wild-type levels confirms the association.

## Related tools

- **MetMiner** (Shiny pipeline used to load the pseudotargeted dataset, run iterative WGCNA, rank mutants by eigenmetabolite, and render metabolite profile figures) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (drives upstream peak picking and data cleaning of the metabolomics data before association mining)
- **MDAtoolkits** (dependency providing statistical analysis, metabolite classification, and enrichment analysis supporting the mining workflow) — https://github.com/ShawnWx2019/MDAtoolkits
- **weighted gene co-expression network analysis** (iterative WGCNA strategy that identifies metabolite modules and eigenmetabolites used to rank mutants) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **R shiny** (framework on which the MetMiner app is built for server deployment and interactive figure export)

## Examples

```
library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- mutant identifiers (including MT_205/max2) correctly map to metabolite abundance columns in the loaded dataset
- the iterative WGCNA run isolates a module (green) whose hub metabolites include glucoraphanin
- mutant_eigenmetabolite_ranking.csv contains exactly top-20 and bottom-20 mutants and flags MT_205/max2 among the bottom mutants for the green-module eigenmetabolite
- the glucoraphanin abundance profile shows higher accumulation in both max2 mutants than wild type, and MAX2 complementation restores wild-type-level glucoraphanin, confirming the negative association
- figures export successfully in pdf/tiff/png/jpg via the MetMiner download button

## Limitations

- MetMiner is flagged as experimental lifecycle and no version numbers are stated for MetMiner, tidyMass, or MDAtoolkits, complicating reproducibility
- No containerization (Docker/Singularity) or persistence guarantee for the hosted Shiny apps is documented; deployment requires an R environment (R >4.1.1 for WGCNA-shinyApp; Linux needs a graphic interface)
- Some in-source fragmentation products may be wrongly annotated as genuine metabolites, with no satisfactory detection/removal solution in the current version

## Evidence

- [abstract] Iterative WGCNA eigenmetabolite ranking and validation workflow: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [discussion] Data-mining validation of F-box gene-metabolite associations: [evidence span withheld — anchor status: not_found]
- [readme] App launch with large file support: "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
- [readme] WGCNA-shinyApp input matrix format requirements: [evidence span withheld — anchor status: not_found]
- [readme] Figure export formats added in MetMiner 1.0.1: "Add download button for figures，format: `pdf`,`tiff`,`png`,`jpg`"
