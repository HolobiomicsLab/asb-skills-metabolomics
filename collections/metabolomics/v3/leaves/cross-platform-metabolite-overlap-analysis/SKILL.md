---
name: cross-platform-metabolite-overlap-analysis
description: Use when you have near-identical feature tables from two different LC–MS
  instrument platforms (e.g. Thermo and Waters) and need to annotate metabolites with
  the PM database and quantify the overlap of reliable annotations across platforms.
when_to_use_negative:
- Your data are from a single LC–MS platform only — the overlap computation requires
  two matched platform datasets
- The feature tables have not been matched to near-identical features across platforms
  (1,901 features in the case study) — unmatched peak lists will not yield comparable
  annotations
- Your goal is general metabolomics data cleaning or peak picking rather than cross-platform
  annotation comparison — use MetMiner's upstream tidyMass-driven data processing
  module instead
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetMiner
  role: Shiny app that runs compound annotation against the PM database and provides
    annotation filtering for each platform dataset
  repo: https://github.com/ShawnWx2019/MetMiner
- name: MDAtoolkits
  role: Companion R package with statistical analysis, metabolite classification,
    and enrichment tools used alongside the annotation workflow
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: tidyMass
  role: Drives upstream data processing (peak picking and data cleaning) producing
    the feature tables used in cross-platform annotation
- name: R shiny
  role: Framework on which the MetMiner app is built for interactive annotation and
    filtering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:6aa6dd6bd77a666acbf268524266910dbdc2ad1e5ff090e2552b70a8bb7fcfcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - R shiny
  - tidyMass
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-platform-metabolite-overlap-analysis

## Summary

Reproduces MetMiner's cross-platform metabolite annotation overlap analysis, in which matched Thermo Scientific and Waters LC–MS feature tables are annotated against the integrated plant-specific metabolite (PM) database and filtered for reliable annotations to quantify platform-consistent metabolite identification.

## When to use

Use when you have near-identical feature tables from two different LC–MS instrument platforms (e.g. Thermo and Waters) and need to annotate metabolites with the PM database and quantify the overlap of reliable annotations across platforms.

## When NOT to use

- Your data are from a single LC–MS platform only — the overlap computation requires two matched platform datasets
- The feature tables have not been matched to near-identical features across platforms (1,901 features in the case study) — unmatched peak lists will not yield comparable annotations
- Your goal is general metabolomics data cleaning or peak picking rather than cross-platform annotation comparison — use MetMiner's upstream tidyMass-driven data processing module instead

## Inputs

- Matched feature table from Thermo Scientific LC–MS platform (1,901 features)
- Matched feature table from Waters LC–MS platform (1,901 near-identical features)
- MetMiner integrated plant-specific metabolite (PM) database

## Outputs

- Per-platform annotated metabolite counts (Thermo and Waters)
- Cross-platform overlap count and percentages (257 shared; 48% of Thermo, 58% of Waters annotations)
- Filtered annotation tables per platform

## How to apply

Launch the MetMiner Shiny app in R with library(tidyverse), library(tidymass), library(MDAtoolkits), library(MetMiner) and run_metMiner(maxRequestSize = 300). Load the two matched feature tables (Thermo and Waters platforms, 1,901 near-identical features) from the MetMiner demo files into the app. Run the compound annotation module against the integrated plant-specific (PM) database, retrieving matched compound records via get_compound_info for each feature, then apply annotation filtering with filter_annotations_massdataset(object, annotate_tbl, method) to retain reliable metabolite annotations in each dataset. Tabulate annotated metabolites per platform and compute the cross-platform overlap; the case-study benchmark is 536 annotated metabolites in Thermo, 447 in Waters, and 257 shared, corresponding to 48% of Thermo and 58% of Waters annotations. Any deviation from these counts flags the annotation or filtering parameters for expert review.

## Related tools

- **MetMiner** (Shiny app that runs compound annotation against the PM database and provides annotation filtering for each platform dataset) — https://github.com/ShawnWx2019/MetMiner
- **MDAtoolkits** (Companion R package with statistical analysis, metabolite classification, and enrichment tools used alongside the annotation workflow) — https://github.com/ShawnWx2019/MDAtoolkits
- **tidyMass** (Drives upstream data processing (peak picking and data cleaning) producing the feature tables used in cross-platform annotation)
- **R shiny** (Framework on which the MetMiner app is built for interactive annotation and filtering)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Exact reproduction of case-study values: 536 annotated metabolites in Thermo, 447 in Waters, 257 shared
- Overlap percentages must compute to 48% of Thermo and 58% of Waters annotations
- Both feature tables must contain 1,901 matched features before annotation
- Any deviation from the reported counts flags the annotation or filtering parameters (method passed to filter_annotations_massdataset) for expert review

## Limitations

- Some suspected in-source fragmentation products are annotated as genuine metabolites, and no satisfactory solution currently exists to detect and remove such redundant annotations
- MetMiner version numbers are not stated in the article, complicating exact reproducibility; the README lists lifecycle as Experimental
- Deviation from expected overlap counts can arise from annotation or filtering parameter choices, requiring expert review

## Evidence

- [results] 536 metabolites in the Thermo Scientific dataset and 447 in the Waters dataset, with 257 metabolites shared: [evidence span withheld — anchor status: not_found]
- [abstract] In two case studies, we validated MetMiner's efficiency in data mining and robustness in metabolite annotation: "In two case studies, we validated MetMiner's efficiency in data mining and robustness in metabolite annotation"
- [methods] filter_annotations_massdataset(object, annotate_tbl, method): "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [readme] run_metMiner(maxRequestSize = 300): "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
