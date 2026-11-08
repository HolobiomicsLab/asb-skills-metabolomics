---
name: pseudotargeted-metabolomics-data-processing
description: Use when the input is a plant pseudotargeted LC-MS metabolomics abundance
  table (e.g. Arabidopsis mutant panel with mutant IDs mapped to metabolite columns)
  and the goal is data cleaning, metabolite annotation, or gene-mutant–metabolite
  association mining such as hub-metabolite WGCNA.
when_to_use_negative:
- Input is already a fully annotated, cleaned feature table that only needs simple
  statistics — MetMiner's upstream tidyMass peak-picking pipeline is unnecessary.
- Non-plant metabolomics data where the integrated plant-specific MS database would
  not improve annotation accuracy.
- 'The WGCNA input matrix uses .csv/.xlsx formats or sample IDs containing spaces
  or special symbols (- @ * & #) or starting with numbers — these are rejected by
  the WGCNA-shinyApp.'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetMiner
  role: 'Integrated R Shiny pipeline providing the entire plant metabolomics workflow:
    data cleaning, metabolite annotation, statistics, and iterative WGCNA data mining'
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Drives upstream data processing (peak picking and data cleaning) with reproducibility,
    traceability, and transparency
- name: MDAtoolkits
  role: Provides the complete set of tools for statistical analysis, metabolite classification,
    and enrichment analysis used downstream
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: weighted gene co-expression network analysis
  role: Iterative WGCNA strategy for efficient biomarker (hub) metabolite screening
    and eigenmetabolite-based mutant ranking
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: R shiny
  role: Framework on which MetMiner is built, allowing server deployment for large-scale
    datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:da57d8100a185771958e45fe67680e590709dc61447ba30503d2ad707e11569f
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# pseudotargeted-metabolomics-data-processing

## Summary

Process and mine large-scale plant pseudotargeted metabolomics datasets using the MetMiner R Shiny pipeline (backed by tidyMass for upstream peak picking/cleaning, a plant-specific MS1/MS2 database for annotation, and iterative WGCNA for biomarker-metabolite screening).

## When to use

Use when the input is a plant pseudotargeted LC-MS metabolomics abundance table (e.g. Arabidopsis mutant panel with mutant IDs mapped to metabolite columns) and the goal is data cleaning, metabolite annotation, or gene-mutant–metabolite association mining such as hub-metabolite WGCNA.

## When NOT to use

- Input is already a fully annotated, cleaned feature table that only needs simple statistics — MetMiner's upstream tidyMass peak-picking pipeline is unnecessary.
- Non-plant metabolomics data where the integrated plant-specific MS database would not improve annotation accuracy.
- The WGCNA input matrix uses .csv/.xlsx formats or sample IDs containing spaces or special symbols (- @ * & #) or starting with numbers — these are rejected by the WGCNA-shinyApp.

## Inputs

- Pseudotargeted metabolomics dataset of plant mutants (metabolite abundance table with sample metadata mapping mutant identifiers, e.g. MT_205/max2, to abundance columns)
- LC-MS raw peak area tables (tidyMass-compatible upstream output)
- Tab-delimited feature-by-sample matrix (.txt/.tsv) for the WGCNA module
- Trait table (qualitative or quantitative, tab-delimited) for WGCNA module-trait association

## Outputs

- Annotated metabolite feature table using the plant-specific MS database
- Module eigenmetabolite values per sample and mutant rankings (e.g. top-20/bottom-20 exported to mutant_eigenmetabolite_ranking.csv)
- Publication-ready figures exportable as pdf/tiff/png/jpg
- Gene-metabolite association results (e.g. F-box protein–metabolite associations)

## How to apply

Install MetMiner and dependencies from GitHub (tidymass via install_tidymass from tidymass.org; ropls, PCAtools, MDAtoolkits, shinyFiles, shinyWidgets, ComplexHeatmap etc. via remotes::install_github), then launch the Shiny app with run_metMiner(maxRequestSize = 300) to enable large-scale file handling. Drive upstream peak picking and data cleaning through tidyMass (inheriting reproducibility, traceability, and transparency), and perform metabolite annotation against the integrated plant-specific mass spectrometry database containing MS1 and MS2 spectra; use MDAtoolkits for statistical analysis, metabolite classification, and enrichment analysis, and helper functions such as QC_boxplot, check_mv, find_noise, and filter_annotations_massdataset during cleaning/filtering. For biomarker discovery, apply the iterative weighted gene co-expression network analysis (WGCNA) module (or the standalone WGCNA-shinyApp, which expects a tab-delimited matrix with features in rows and samples in columns and rejects spaces/special symbols in sample IDs and pure-numeric feature IDs), computing module eigenmetabolite values per sample to rank mutants. Downstream, extract individual metabolite abundances (e.g. glucoraphanin) and export publication-ready figures in pdf/tiff/png/jpg via the app's download button. Judge success by whether the pipeline's annotation, module identification, and mutant rankings reproduce the expected associations (e.g. MT_205/max2 flagged among bottom mutants for the green-module eigenmetabolite).

## Related tools

- **MetMiner** (Integrated R Shiny pipeline providing the entire plant metabolomics workflow: data cleaning, metabolite annotation, statistics, and iterative WGCNA data mining) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Drives upstream data processing (peak picking and data cleaning) with reproducibility, traceability, and transparency)
- **MDAtoolkits** (Provides the complete set of tools for statistical analysis, metabolite classification, and enrichment analysis used downstream) — https://github.com/ShawnWx2019/MDAtoolkits
- **weighted gene co-expression network analysis** (Iterative WGCNA strategy for efficient biomarker (hub) metabolite screening and eigenmetabolite-based mutant ranking) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **R shiny** (Framework on which MetMiner is built, allowing server deployment for large-scale datasets)

## Examples

```
library(MetMiner); library(tidymass); library(MDAtoolkits); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Sample metadata correctly maps mutant identifiers (e.g. MT_205/max2) to metabolite abundance columns before analysis.
- Iterative WGCNA isolates a coherent module (e.g. green module) whose hub metabolites include known glucosinolates such as glucoraphanin.
- Eigenmetabolite ranking flags a validated mutant (MT_205/max2) among the bottom mutants for the green-module eigenmetabolite, consistent with the published negative MAX2–glucoraphanin association.
- Metabolite annotations are produced against the plant-specific MS1/MS2 database and pass the pipeline's annotation filtering (filter_annotations_massdataset).
- Figures export successfully in pdf/tiff/png/jpg via the MetMiner download button (added in version 1.0.1).

## Limitations

- There is currently no satisfactory solution to detect and remove in-source fragmentation products wrongly annotated as genuine metabolites; this is a focus for the next MetMiner version.
- No release version numbers, containerization (Docker/Singularity), or persistence guarantees for the hosted Shiny apps are stated; the pipeline lifecycle is marked Experimental.
- The WGCNA-shinyApp requires a graphic interface on Linux and only accepts tab-delimited files with strict sample ID formatting constraints.

## Evidence

- [intro] The upstream data processing (peak picking and data cleaning) is driven by tidyMass, inheriting the advantages of reproducibility, traceability, and transparency.: "The upstream data processing (peak picking and data cleaning) is driven by tidyMass, inheriting the advantages of reproducibility, traceability, and transparency."
- [intro] Plant-specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy.: "Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [intro] An iterative weighted gene co-expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency.: "(iii) An iterative weighted gene co‐expression network analysis (WGCNA) strategy has been adopted to improve data mining efficiency."
- [readme] Launch the MetMiner Shiny application via run_metMiner(maxRequestSize = 300).: "run_metMiner(maxRequestSize = 300)"
- [readme] WGCNA input format constraints from README: tab-delimited, strict IDs.: [evidence span withheld — anchor status: external_doc]
- [discussion] In-source fragmentation limitation acknowledged in discussion.: "Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner."
