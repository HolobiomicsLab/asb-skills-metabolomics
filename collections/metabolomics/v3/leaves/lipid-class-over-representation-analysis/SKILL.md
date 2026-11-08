---
name: lipid-class-over-representation-analysis
description: Use when a differential expression result table (e.g., the deSp_multiGroup
  output from a multi-group lipidomics analysis) contains significant lipid species
  that should be tested for over-representation in lipid characteristics such as lipid
  class.
when_to_use_negative:
- The input is a raw lipidomics profile with no differential expression results —
  run the LipidSigR Differential Expression workflow first to obtain significant lipid
  species.
- The goal is Lipid Set Enrichment Analysis (LSEA) over ranked lipid sets rather than
  ORA on a list of significant species.
- The task targets the Network or Machine learning workflows, which LipidSigR restricts
  to two-group data.
edam_topics:
- http://edamontology.org/topic_3956
tools:
- name: LipidSigR
  role: Provides the enrichment_ora function and the lipid characteristic set (lipid
    class, chain length, unsaturation, hydroxyl groups, fatty acid composition) used
    to run ORA and render the bar plot
  repo: https://github.com/BioinfOMICS/LipidSigR
- name: R
  role: Runtime environment; LipidSigR requires R 4.3.0 or later
- name: dplyr
  role: Column handling on the deSp_multiGroup differential expression results table
    before running ORA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf047
    title: 'LipidSigR: a R-based solution for integrated lipidomics data analysis
      and visualization'
  source_content_hash: sha256:310cb1e8f57db3a7dc5bb72215ee4d6332a61cf019ab6aa33f2169c761a45ae9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfOMICS/LipidSigR
  tools:
  - LipidSigR
  - R
  - dplyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3956
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lipid-class-over-representation-analysis

## Summary

Performs Over Representation Analysis (ORA) of significant lipid species from differential expression results using LipidSigR, classifying species as up- or down-regulated and testing their enrichment in lipid characteristic categories (e.g., lipid class), visualized as a bar plot.

## When to use

Use when a differential expression result table (e.g., the deSp_multiGroup output from a multi-group lipidomics analysis) contains significant lipid species that should be tested for over-representation in lipid characteristics such as lipid class.

## When NOT to use

- The input is a raw lipidomics profile with no differential expression results — run the LipidSigR Differential Expression workflow first to obtain significant lipid species.
- The goal is Lipid Set Enrichment Analysis (LSEA) over ranked lipid sets rather than ORA on a list of significant species.
- The task targets the Network or Machine learning workflows, which LipidSigR restricts to two-group data.

## Inputs

- deSp_multiGroup differential expression results table (significant lipid species with log2 fold change) from the LipidSigR Differential Expression workflow
- LipidSigR-processed SummarizedExperiment-derived lipidomics data from the upstream profiling/de analysis
- LipidSigR lipid characteristic set (lipid class, chain length, unsaturation, hydroxyl groups, fatty acid composition)

## Outputs

- ORA enrichment result object/table from enrichment_ora listing enriched lipid-characteristic terms per up-/down-regulated category
- ora_barplot.png — bar plot of top enriched lipid-class terms for up- and down-regulated categories

## How to apply

Load the deSp_multiGroup differential expression results table produced by the upstream multi-group analysis, handling columns with dplyr as needed. Call LipidSigR's enrichment_ora function on the significant lipid species, splitting them into 'up-regulated' or 'down-regulated' categories based on their log2 fold change, and test enrichment against the package's lipid characteristic set (lipid class, chain length, unsaturation, hydroxyl groups, and fatty acid composition). Render the ORA bar plot showing the top enriched lipid-class terms for the up- and down-regulated categories as returned by enrichment_ora, and write it to ora_barplot.png. ORA is one of the two enrichment approaches in LipidSigR's Enrichment workflow, the other being Lipid Set Enrichment Analysis (LSEA); consult vignette('enrichment') for the full procedure and vignette('case_multiGroup') for the multi-group case study.

## Related tools

- **LipidSigR** (Provides the enrichment_ora function and the lipid characteristic set (lipid class, chain length, unsaturation, hydroxyl groups, fatty acid composition) used to run ORA and render the bar plot) — https://github.com/BioinfOMICS/LipidSigR
- **R** (Runtime environment; LipidSigR requires R 4.3.0 or later)
- **dplyr** (Column handling on the deSp_multiGroup differential expression results table before running ORA)

## Examples

```
library(LipidSigR); ora_result <- enrichment_ora(input = deSp_multiGroup_result, characteristic = "class")
```

## Evaluation signals

- ora_barplot.png exists and is a readable PNG image file.
- The bar plot contains the expected up-/down-regulated enriched-term bars for the multi-group case study data.
- Significant lipid species are split into exactly two categories ('up-regulated' and 'down-regulated') based on log2 fold change.
- Enriched terms are drawn from the package's lipid characteristics (lipid class, chain length, unsaturation, hydroxyl groups, fatty acid composition).
- Results are consistent with the procedure documented in vignette('enrichment') and the multi-group case example vignette('case_multiGroup').

## Limitations

- ORA uses only the significant lipid species list; LipidSigR's Enrichment workflow offers the alternative LSEA approach for a priori-defined lipid sets with concordant differences between biological states.
- LipidSigR requires R 4.3.0 or later and, on Windows, Windows 10 or later; several dependencies (e.g., fgsea, gatom, mixOmics, rgoslin via BiocManager) must be installed if missing.
- Only two-group data can conduct the Network and Machine learning workflows in LipidSigR, so multi-group ORA cannot be extended into those downstream analyses.

## Evidence

- [readme] The enrichment workflow provides two main approaches: 'Over Representation Analysis (ORA)' and 'Lipid Set Enrichment Analysis (LSEA)' to illustrates significant lipid species enriched in the categories of lipid class: "The enrichment workflow provides two main approaches: 'Over Representation Analysis (ORA)' and 'Lipid Set Enrichment Analysis (LSEA)' to illustrates significant lipid species enriched in the"
- [readme] Each section offers unique aspects to analyzing lipidome profiling data based on various characteristics, including lipid class, chain length, unsaturation, hydroxyl groups, and fatty acid composition.: "Each section offers unique aspects to analyzing lipidome profiling data based on 
various characteristics, including lipid class, chain length, unsaturation, 
hydroxyl groups, and fatty acid"
- [intro] This documentation will show the complete procedure for analyzing multi-group data.: "This documentation will show the complete procedure for analyzing
multi-group data."
- [readme] You need to have R 4.3.0 or a later version installed for running LipidSigR.: "You need to have R 4.3.0 or a later version installed for running LipidSigR."
