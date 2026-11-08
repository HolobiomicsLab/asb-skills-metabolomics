---
name: eigenmetabolite-mutant-ranking
description: Use when you have a metabolomics feature matrix across many mutant lines
  plus sample metadata, and need to rank mutants on a co-abundance module's eigenmetabolite
  to nominate lines for validation of a metabolite–gene association.
when_to_use_negative:
- Input is raw LC-MS vendor data rather than a processed/normalized abundance table
  — MetMiner's upstream peak picking and data cleaning (driven by tidyMass) must be
  run first
- 'The WGCNA-shinyApp input is not tab-delimited (.txt/.tsv) or uses pure numbers
  as feature IDs or sample IDs with spaces or special symbols like - @ * & #, which
  the app rejects'
- The goal is transcript/protein co-expression module eigengene ranking with a standalone
  tool and no need for metabolomics-specific annotation — the generic WGCNA-shinyApp
  or plain R WGCNA suffices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetMiner
  role: R Shiny pipeline that runs iterative WGCNA on metabolomics data, computes
    module eigenmetabolites, ranks mutants, and exports figures/results via download
    buttons
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Drives upstream peak picking and data cleaning that produce the normalized
    abundance matrix used as WGCNA input
- name: MDAtoolkits
  role: Statistical analysis toolkit installed alongside MetMiner for downstream metabolomics
    statistics
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: weighted gene co-expression network analysis
  role: Module-detection method whose iterative variant defines the metabolite modules
    whose eigenmetabolite is the ranking metric
- name: WGCNA-shinyApp
  role: Standalone Shiny app for the WGCNA step, taking an expression/peak-area matrix
    plus trait table as tab-delimited input
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:6910034d0f9fe5de5ebe13e7a840d87a6424fffff0d348b5c2a04f8cbc4497ad
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# eigenmetabolite-mutant-ranking

## Summary

Ranks mutants by the module eigenmetabolite (first principal component) of a WGCNA metabolite module, computed on a pseudotargeted metabolomics matrix, to nominate mutant lines (e.g. F-box protein mutants like MT_205/max2) that perturb hub metabolites such as glucoraphanin.

## When to use

Use when you have a metabolomics feature matrix across many mutant lines plus sample metadata, and need to rank mutants on a co-abundance module's eigenmetabolite to nominate lines for validation of a metabolite–gene association.

## When NOT to use

- Input is raw LC-MS vendor data rather than a processed/normalized abundance table — MetMiner's upstream peak picking and data cleaning (driven by tidyMass) must be run first
- The WGCNA-shinyApp input is not tab-delimited (.txt/.tsv) or uses pure numbers as feature IDs or sample IDs with spaces or special symbols like - @ * & #, which the app rejects
- The goal is transcript/protein co-expression module eigengene ranking with a standalone tool and no need for metabolomics-specific annotation — the generic WGCNA-shinyApp or plain R WGCNA suffices

## Inputs

- Pseudotargeted metabolomics abundance matrix (e.g. peak area per metabolite feature per sample)
- Sample metadata mapping mutant identifiers (e.g. MT_205/max2) to metabolite abundance columns
- WGCNA module assignment (e.g. green module) containing hub metabolites of interest

## Outputs

- mutant_eigenmetabolite_ranking.csv with top-20 and bottom-20 mutants ranked on the module eigenmetabolite metric
- Per-sample module eigenmetabolite values
- Accumulation profile figure for the target metabolite (exportable as pdf/tiff/png/jpg via the MetMiner download button)

## How to apply

Load MetMiner, tidyMass, and MDAtoolkits (installed from GitHub, with ropls and PCAtools as dependencies) and launch the Shiny app with run_metMiner(maxRequestSize = 300) to handle large files. Load the pseudotargeted metabolomics dataset and confirm sample metadata maps mutant identifiers (e.g. MT_205/max2) to abundance columns. Run the iterative weighted gene co-expression network analysis (WGCNA) module to isolate the module of interest (e.g. the green module containing hub glucosinolate metabolites including glucoraphanin); note the WGCNA input must be a tab-delimited table with features in rows and sample IDs in columns, IDs free of spaces/special symbols and not starting with numbers. Compute the module eigenmetabolite per sample (PC1 summary of module member abundances), rank mutants on this metric, and export the top-20 and bottom-20 mutants to mutant_eigenmetabolite_ranking.csv. Finally, extract the target metabolite's abundance across all mutant lines to check that top/bottom-ranked mutants show the expected accumulation phenotype, as in the negative MAX2–glucoraphanin association where max2 mutants fell in the bottom group.

## Related tools

- **MetMiner** (R Shiny pipeline that runs iterative WGCNA on metabolomics data, computes module eigenmetabolites, ranks mutants, and exports figures/results via download buttons) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Drives upstream peak picking and data cleaning that produce the normalized abundance matrix used as WGCNA input)
- **MDAtoolkits** (Statistical analysis toolkit installed alongside MetMiner for downstream metabolomics statistics) — https://github.com/ShawnWx2019/MDAtoolkits
- **weighted gene co-expression network analysis** (Module-detection method whose iterative variant defines the metabolite modules whose eigenmetabolite is the ranking metric)
- **WGCNA-shinyApp** (Standalone Shiny app for the WGCNA step, taking an expression/peak-area matrix plus trait table as tab-delimited input) — https://github.com/ShawnWx2019/WGCNA-shinyApp

## Examples

```
library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- The exported ranking CSV contains exactly the top-20 and bottom-20 mutants ordered by module eigenmetabolite value
- The target mutant (e.g. MT_205/max2) is flagged among the bottom mutants for the green-module eigenmetabolite
- The extracted metabolite profile (e.g. glucoraphanin) shows reduced accumulation in max2 mutants relative to wild type, and complementation of MAX2 restores wild-type abundance
- WGCNA input passed format validation: features in rows, sample IDs in columns, no spaces/special symbols in sample IDs, tab-delimited .txt/.tsv
- Figures render in the Shiny app and export successfully in pdf/tiff/png/jpg (download buttons added in MetMiner v1.0.1)

## Limitations

- No version number, release DOI, or containerization (Docker/Singularity) is stated for MetMiner, tidyMass, or MDAtoolkits, so exact reproducibility of the ranking is not guaranteed
- README flags the app as 'Experimental' lifecycle and Linux deployment requires a graphic interface
- Eigenmetabolite ranking is correlational; the MAX2–glucoraphanin association required genetic validation (mutant + complementation) before it could be called negative

## Evidence

- [abstract] Iterative WGCNA isolates module and ranks mutants: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [discussion] Validation via F-box protein associations: [evidence span withheld — anchor status: not_found]
- [readme] Launch command with large file support: "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
- [readme] Input format constraints for WGCNA app: [evidence span withheld — anchor status: not_found]
- [readme] Figure export formats added in v1.0.1: "Add download button for figures，format: `pdf`,`tiff`,`png`,`jpg`;"
