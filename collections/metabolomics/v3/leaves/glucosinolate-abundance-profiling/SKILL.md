---
name: glucosinolate-abundance-profiling
description: Use when a pseudotargeted metabolomics abundance table (mutant IDs like
  MT_205/max2 mapped to metabolite columns) needs per-mutant glucosinolate accumulation
  profiles to validate hub-metabolite associations flagged by module eigenmetabolite
  screening.
when_to_use_negative:
- Input is raw LC-MS vendor data (mzML/Raw) needing peak picking and annotation rather
  than a processed abundance table — start from the upstream tidyMass-driven data
  processing module instead
- The metabolite of interest is not represented in the pseudotargeted panel or the
  green module hub metabolite set — glucosinolate profiling here depends on glucoraphanin
  being quantified
- Mutant identifiers in the metadata do not map unambiguously to abundance columns
  (e.g. MT_205/max2 naming broken), since eigenmetabolite ranking and genotype-specific
  profiling both rely on that mapping
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMiner
  role: Shiny pipeline used to load the pseudotargeted dataset, run iterative WGCNA,
    and render/export the glucoraphanin accumulation profile figure
  repo: https://github.com/ShawnWx2019/MetMiner
- name: weighted gene co-expression network analysis
  role: Iterative WGCNA strategy isolating the green module of hub glucosinolate metabolites
    and computing eigenmetabolite values used to rank mutants
  repo: https://github.com/ShawnWx2019/WGCNA-shinyApp
- name: tidyMass
  role: Drives upstream peak picking and data cleaning that produces the reproducible
    abundance table the profiling step consumes
  repo: https://github.com/ShawnWx2019/MetMiner
- name: MDAtoolkits
  role: Statistical analysis toolkit installed as a required dependency for downstream
    metabolite statistics during profiling
  repo: https://github.com/ShawnWx2019/MetMiner
- name: R shiny
  role: Framework on which the MetMiner app runs, deployable on servers for large-scale
    datasets
  repo: https://github.com/ShawnWx2019/MetMiner
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:cda13885a8e8f305dc93b1cbfff35c1d54ea5411431e03d72db082e046114a4c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# glucosinolate-abundance-profiling

## Summary

Profiles glucosinolate (e.g. glucoraphanin) abundance across an Arabidopsis mutant panel from a pseudotargeted metabolomics dataset, and cross-checks the metabolite-level pattern against green-module eigenmetabolite rankings from iterative WGCNA to confirm gene–metabolite associations such as the negative MAX2–glucoraphanin link.

## When to use

Use when a pseudotargeted metabolomics abundance table (mutant IDs like MT_205/max2 mapped to metabolite columns) needs per-mutant glucosinolate accumulation profiles to validate hub-metabolite associations flagged by module eigenmetabolite screening.

## When NOT to use

- Input is raw LC-MS vendor data (mzML/Raw) needing peak picking and annotation rather than a processed abundance table — start from the upstream tidyMass-driven data processing module instead
- The metabolite of interest is not represented in the pseudotargeted panel or the green module hub metabolite set — glucosinolate profiling here depends on glucoraphanin being quantified
- Mutant identifiers in the metadata do not map unambiguously to abundance columns (e.g. MT_205/max2 naming broken), since eigenmetabolite ranking and genotype-specific profiling both rely on that mapping

## Inputs

- Pseudotargeted metabolomics abundance table of Arabidopsis mutants (metabolites x mutant lines, e.g. MT_205/max2 columns)
- Sample metadata mapping mutant identifiers to metabolite abundance columns
- Green-module eigenmetabolite ranking from iterative WGCNA (top-20/bottom-20 mutants)

## Outputs

- Glucoraphanin accumulation profile across all mutant lines (figure exportable as pdf/tiff/png/jpg)
- mutant_eigenmetabolite_ranking.csv (top-20 and bottom-20 mutants by green-module eigenmetabolite)
- Validated MAX2–glucoraphanin association (negative direction, restored by complementation)

## How to apply

Install MetMiner, tidyMass and MDAtoolkits from GitHub (with ropls and PCAtools as required dependencies), then launch the Shiny app via run_metMiner(maxRequestSize = 300) to handle large-scale files. Load the pseudotargeted metabolomics dataset of Arabidopsis mutants and confirm that sample metadata maps mutant identifiers (including MT_205/max2) to metabolite abundance columns. After the iterative WGCNA module isolates the green module containing hub glucosinolate metabolites, compute module eigenmetabolite values per sample, rank mutants on the eigenmetabolite metric, and export the top-20 and bottom-20 mutants to mutant_eigenmetabolite_ranking.csv. Extract glucoraphanin abundance across all mutant lines and render the accumulation profile figure, exporting in pdf/tiff/png/jpg via the MetMiner download button (available since v1.0.1). Validation requires that the ranking flags MT_205/max2 among the bottom mutants for the green-module eigenmetabolite, that glucoraphanin is reduced in max2 relative to wild type, and that complementary MAX2 expression restores glucoraphanin to the wild-type level, confirming the negative association.

## Related tools

- **MetMiner** (Shiny pipeline used to load the pseudotargeted dataset, run iterative WGCNA, and render/export the glucoraphanin accumulation profile figure) — https://github.com/ShawnWx2019/MetMiner
- **weighted gene co-expression network analysis** (Iterative WGCNA strategy isolating the green module of hub glucosinolate metabolites and computing eigenmetabolite values used to rank mutants) — https://github.com/ShawnWx2019/WGCNA-shinyApp
- **tidyMass** (Drives upstream peak picking and data cleaning that produces the reproducible abundance table the profiling step consumes) — https://github.com/ShawnWx2019/MetMiner
- **MDAtoolkits** (Statistical analysis toolkit installed as a required dependency for downstream metabolite statistics during profiling) — https://github.com/ShawnWx2019/MetMiner
- **R shiny** (Framework on which the MetMiner app runs, deployable on servers for large-scale datasets) — https://github.com/ShawnWx2019/MetMiner

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- MT_205/max2 appears among the bottom mutants of the green-module eigenmetabolite ranking in mutant_eigenmetabolite_ranking.csv
- The glucoraphanin accumulation profile shows higher glucoraphanin in both max2 mutants versus wild type
- Complementation (MAX2 expressed in max2-1) restores glucoraphanin abundance to the wild-type level in the profile figure
- Profile figure exports successfully via the MetMiner download button in pdf/tiff/png/jpg (feature added in v1.0.1)
- Sample metadata check passes: mutant identifiers including MT_205/max2 map correctly to metabolite abundance columns before ranking

## Limitations

- MetMiner and its companion packages state no version numbers in the article and had no release DOI at publication time; v1.0.0→v1.0.1 changes (figure download button, classification bug fix) affect figure export availability
- No containerization (Docker/Singularity) or persistence guarantee is provided for the hosted Shiny apps, complicating reproducible re-runs of the profiling workflow
- The validation is specific to the negative MAX2–glucoraphanin association observed in Arabidopsis mutants; direction of associations for other gene–metabolite pairs may differ and must be checked independently

## Evidence

- [abstract] iterative WGCNA screens hub metabolites: "we propose an iterative weighted gene co-expression network analysis strategy for efficient biomarker metabolite screening"
- [discussion] gene–metabolite associations validated: [evidence span withheld — anchor status: not_found]
- [readme] figure export formats in MetMiner v1.0.1: "Add download button for figures，format: `pdf`,`tiff`,`png`,`jpg`;"
- [readme] launching the MetMiner Shiny app: "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
