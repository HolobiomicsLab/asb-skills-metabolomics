---
name: wgcna-module-eigenmetabolite-screening
description: Use when analyzing a pseudotargeted metabolomics feature/peak-area table
  (metabolites × samples) across a mutant or treatment panel to find co-abundance
  modules and rank samples by module eigenmetabolite for biomarker metabolite screening.
when_to_use_negative:
- Input is raw LC-MS data (mzML etc.) rather than a peak table — upstream peak picking
  and data cleaning by tidyMass within MetMiner must be run first
- Input file is .csv, .xlsx or .xls — the WGCNA-shinyApp only accepts tab-delimited
  .txt/.tsv files, so convert the format first
- The analysis goal is metabolite annotation from spectra rather than co-abundance
  network mining — use MetMiner's annotation pipeline with the plant-specific MS database
  instead
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMiner
  role: Hosts the iterative WGCNA module within its Shiny app for module detection,
    eigenmetabolite computation, mutant ranking, and figure export
  repo: https://github.com/ShawnWx2019/MetMiner
- name: WGCNA-shinyApp
  role: Standalone Shiny app for expression-matrix filtering (noise removal, MAD/VAR-based
    top-N filtering) and WGCNA network construction with optional trait data
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: tidyMass
  role: Provides the upstream peak picking and data cleaning that produce the metabolite
    abundance matrix fed into WGCNA
- name: MDAtoolkits
  role: Required dependency providing statistical analysis tools used alongside the
    MetMiner WGCNA workflow
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: R shiny
  role: Framework on which MetMiner is built, enabling server deployment for large-scale
    datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:2e9415b5170b1deba5240097ead7ff296165ddee32df9c6528b147feebf5bae0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - R shiny
  - WGCNA-shinyApp
  - tidyMass
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# wgcna-module-eigenmetabolite-screening

## Summary

An iterative weighted gene co-expression network analysis (WGCNA) strategy, delivered through MetMiner's Shiny interface, that clusters metabolite abundance features into modules and uses module eigenmetabolites to rank mutants and screen for biomarker metabolites such as hub glucosinolates.

## When to use

Use when analyzing a pseudotargeted metabolomics feature/peak-area table (metabolites × samples) across a mutant or treatment panel to find co-abundance modules and rank samples by module eigenmetabolite for biomarker metabolite screening.

## When NOT to use

- Input is raw LC-MS data (mzML etc.) rather than a peak table — upstream peak picking and data cleaning by tidyMass within MetMiner must be run first
- Input file is .csv, .xlsx or .xls — the WGCNA-shinyApp only accepts tab-delimited .txt/.tsv files, so convert the format first
- The analysis goal is metabolite annotation from spectra rather than co-abundance network mining — use MetMiner's annotation pipeline with the plant-specific MS database instead

## Inputs

- Tab-delimited expression/abundance matrix (.txt or .tsv): metabolite/gene/protein IDs in rows, sample IDs in columns (peak area for metabolomics)
- Optional trait table: quantitative or qualitative (binary-encoded) sample traits
- Sample metadata mapping mutant identifiers (e.g. MT_205/max2) to abundance columns

## Outputs

- Co-abundance modules (e.g. the green module containing hub glucosinolate metabolites including glucoraphanin)
- Module eigenmetabolite values per sample and mutant ranking (e.g. top-20/bottom-20 mutants exported to mutant_eigenmetabolite_ranking.csv)
- Publication-ready figures of metabolite accumulation profiles, exportable as pdf/tiff/png/jpg via the MetMiner download button

## How to apply

Prepare the input as a tab-delimited matrix (.txt/.tsv) with metabolite IDs in rows and sample IDs in columns (peak area values; sample IDs must not contain spaces or special symbols like - @ * & # and must not start with numbers, and IDs cannot be pure numbers). Clean the data before network construction: remove low-abundance features (e.g. those below cutoff in a set percentage of samples) and optionally filter to the top-N features by median absolute deviation (MAD) or variance (VAR), following WGCNA FAQ guidance. Run the iterative WGCNA module inside MetMiner's Shiny app (launched with run_metMiner(maxRequestSize = 300)) or the standalone WGCNA-shinyApp, optionally supplying a trait table (quantitative or binary qualitative) to relate modules to phenotypes. Iterate to isolate the module containing the hub metabolites of interest, compute the module eigenmetabolite per sample, rank mutants on that metric (e.g. export top-20/bottom-20 mutants to CSV), and extract abundances of individual metabolites (e.g. glucoraphanin) to connect module membership with gene-metabolite associations such as the negative MAX2–glucoraphanin link.

## Related tools

- **MetMiner** (Hosts the iterative WGCNA module within its Shiny app for module detection, eigenmetabolite computation, mutant ranking, and figure export) — https://github.com/ShawnWx2019/MetMiner
- **WGCNA-shinyApp** (Standalone Shiny app for expression-matrix filtering (noise removal, MAD/VAR-based top-N filtering) and WGCNA network construction with optional trait data) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **tidyMass** (Provides the upstream peak picking and data cleaning that produce the metabolite abundance matrix fed into WGCNA)
- **MDAtoolkits** (Required dependency providing statistical analysis tools used alongside the MetMiner WGCNA workflow) — https://github.com/ShawnWx2019/MDAtoolkits
- **R shiny** (Framework on which MetMiner is built, enabling server deployment for large-scale datasets)

## Examples

```
library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- The eigenmetabolite ranking flags the expected mutant (e.g. MT_205/max2) among the bottom mutants for the green-module eigenmetabolite
- The isolated module contains known hub metabolites of the pathway of interest (e.g. glucosinolate hub metabolites including glucoraphanin)
- Individual metabolite abundance profiles agree with the module signal (e.g. reduced glucoraphanin in max2, restored to wild-type by MAX2 complementation)
- Input passes format validation: tab-delimited file, non-numeric feature IDs, sample IDs without spaces or special symbols (- @ * & #) and not starting with numbers
- Feature table was filtered as specified: low-abundance features (below cutoff in a high percentage of samples) removed and top-N features by MAD or VAR retained

## Limitations

- Repository lifecycle is marked 'Experimental'; MetMiner, tidyMass and MDAtoolkits have no stated version numbers or release DOIs in the article, complicating reproducibility
- WGCNA-shinyApp on Linux requires a machine with a graphical interface; CSV/Excel inputs and pure-numeric IDs are rejected
- WGCNA FAQ guidance notes that low-expressed features reflect noise and correlation-based modules are only meaningful after transformation (e.g. vst or log10(x+1)) and filtering — skipping cleaning degrades module quality

## Evidence

- [abstract] iterative WGCNA strategy for biomarker screening: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [intro] iterative WGCNA adopted to improve data mining: "(iii) An iterative weighted gene co‐expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency."
- [discussion] gene-metabolite associations validated from pseudotargeted dataset: [evidence span withheld — anchor status: not_found]
- [readme] input matrix format requirements: [evidence span withheld — anchor status: not_found]
- [readme] feature filtering by MAD or variance: [evidence span withheld — anchor status: external_doc]
- [readme] noise removal threshold guidance: [evidence span withheld — anchor status: external_doc]
- [readme] peak area as metabolomics input: [evidence span withheld — anchor status: external_doc]
