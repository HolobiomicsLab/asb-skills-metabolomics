---
name: aggregate-statistics-meta-analysis
description: Use when input consists of published per-study aggregate statistics (p-value
  and fold-change) for metabolites across clinical studies, and compounds must be
  harmonized across synonym identifiers before combining.
when_to_use_negative:
- You have access to individual-level (raw intensity) data across studies — then a
  full statistical meta-analysis in a tool like MetaboAnalyst on the primary data
  is the appropriate route, as in the PR000293 comparison
- The input already contains a single harmonized compound list with consistent identifiers
  and no synonym/duplicate problem to resolve
- Compounds across studies lack any p-value or fold-change reporting, since the method
  operates on aggregate statistics only
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: Easy-Amanida
  role: Shiny web app that runs the meta-analysis of aggregate data (p-value, fold-change)
    with a workflow to optimize naming combination; the app link is http://brui.shinyapps.io/easy-amanida
  repo: https://github.com/mariallr/easy-amanida
- name: amanida
  role: R package performing the vote-counting and quantitative combination of p-values
    and fold-changes
  repo: https://github.com/mariallr/easy-amanida
- name: webchem
  role: Harmonizes heterogeneous compound names/IDs (common names, InChIKeys, SMILES)
    onto a common chemical nomenclature before combining
  repo: https://github.com/mariallr/easy-amanida
- name: Shiny
  role: Framework in which the Easy-Amanida app is implemented for interactive upload
    and analysis
- name: R
  role: Language in which amanida and webchem are run and results are exported to
    CSV
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_001
  source_papers:
  - doi: 10.1002/jrsm.1713
    title: 'Easy‐Amanida: An R Shiny application for the meta‐analysis of aggregate
      results in clinical metabolomics using Amanida and Webchem'
  source_content_hash: sha256:b7df537011c7671514bf1aa43086ca3702733d7b5f4c34769a79c2e9695afd08
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/easy-amanida
  repo_url: https://github.com/mariallr/easy-amanida
  tools:
  - Easy-Amanida
  - R
  - Shiny
  - amanida
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# aggregate-statistics-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Meta-analysis of aggregate metabolomics results (p-values and fold-changes from multiple clinical studies) using the Easy-Amanida Shiny app, which wraps the amanida R package and harmonizes heterogeneous compound names with webchem. It supports both qualitative vote-counting and quantitative combination of published statistics without access to individual-level data.

## When to use

Use when input consists of published per-study aggregate statistics (p-value and fold-change) for metabolites across clinical studies, and compounds must be harmonized across synonym identifiers before combining.

## When NOT to use

- You have access to individual-level (raw intensity) data across studies — then a full statistical meta-analysis in a tool like MetaboAnalyst on the primary data is the appropriate route, as in the PR000293 comparison
- The input already contains a single harmonized compound list with consistent identifiers and no synonym/duplicate problem to resolve
- Compounds across studies lack any p-value or fold-change reporting, since the method operates on aggregate statistics only

## Inputs

- Table of per-study aggregate statistics: compound identifier, p-value, and fold-change (e.g. Supplementary Table 1 PDAC, Supplementary Table 2 PR000293 lung cancer blood data from Metabolomics Workbench, Supplementary Table 3 colorectal cancer)
- Mixed-type compound identifiers (common names, InChIKeys, SMILES) when the 'Include compounds IDs' harmonization option is enabled

## Outputs

- Per-compound combined meta-analysis results table (combined p-value, log2 fold-change, trend direction, number of contributing studies), exportable as CSV (e.g. amanida_metaanalysis_results.csv, vote_counting_results.csv, harmonization_results.csv)
- Vote-counting summary: number of studies with consistent trend per metabolite, up-/down-regulated significant compound tallies, and inconsistent-trend flags

## How to apply

Load the dataset of aggregate statistics (p-value and fold-change per compound per study, e.g. PDAC Supplementary Table 1 from 24 clinical studies) into R and launch the Easy-Amanida Shiny app. Choose the qualitative amanida vote-counting mode (+1 for up-regulated, -1 for down-regulated per study) or the quantitative mode combining p-values and log2 fold-changes, relying on the webchem-backed naming harmonization to unify compound synonyms; activate the 'Include compounds IDs' option when identifiers are mixed types (common names, InChIKeys, SMILES) so they are standardized to a common ID, collapsing duplicates (e.g. 133 IDs to 120 unique). Flag metabolites with inconsistent regulation trends across studies and exclude them from the significant-compound tally. Judge significance with the paper's thresholds: quantitative calls require combined p-value < 0.05 and fold-change > 2; vote-counting validity is checked by reproducing reported recurrence counts (e.g. glutamine down-regulated in 7 studies, histidine in 5, three bile acids in 3 each).

## Related tools

- **Easy-Amanida** (Shiny web app that runs the meta-analysis of aggregate data (p-value, fold-change) with a workflow to optimize naming combination; the app link is http://brui.shinyapps.io/easy-amanida) — https://github.com/mariallr/easy-amanida
- **amanida** (R package performing the vote-counting and quantitative combination of p-values and fold-changes) — https://github.com/mariallr/easy-amanida
- **webchem** (Harmonizes heterogeneous compound names/IDs (common names, InChIKeys, SMILES) onto a common chemical nomenclature before combining) — https://github.com/mariallr/easy-amanida
- **Shiny** (Framework in which the Easy-Amanida app is implemented for interactive upload and analysis)
- **R** (Language in which amanida and webchem are run and results are exported to CSV)

## Examples

```
shiny::runGitHub("easy-amanida", "mariallr")  # then upload the aggregate-statistics table and run amanida in the app
```

## Evaluation signals

- Quantitative mode reproduces the PDAC benchmark: 311 metabolites combined, 38 compounds in more than one study, 35 significant metabolites (p<0.05, FC>2), and Table 2 entries (Histidine combined p 0.023, N 9500; Valine FC 160 up; Tryptophan FC 150 up)
- Qualitative vote-counting reproduces reported counts: glutamine in 7 studies (down), histidine in 5 (down), tauroursodeoxycholic/taurocholic/glycocholic acid in 3 studies each (up), and 8 up / 12 down significant compounds after excluding the 15 inconsistent-trend metabolites
- With 'Include compounds IDs' enabled on the colorectal dataset, unique identifiers drop from 133 to 120 and vote counts rise to 3 for both hippuric acid and indole-3-acetic acid
- On PR000293, consistency with MetaboAnalyst: three metabolites significant in both analyses, salicylic acid significant in Easy-Amanida with inconsistent trends
- Exported CSV tables contain per-compound combined statistics with no unresolved synonym duplicates

## Limitations

- Operates only on aggregate statistics (p-value, fold-change); it cannot re-analyze raw data, so conclusions depend on the statistics reported by original studies
- Metabolites with inconsistent regulation trends across studies must be flagged and excluded from the significant tally, so discordant evidence is not resolvable by this method
- Compound naming synonyms across studies hinder meta-analysis unless webchem harmonization (optionally with 'Include compounds IDs') is applied; mixed identifier types otherwise inflate unique-ID counts and split vote counts

## Evidence

- [abstract] Easy-Amanida combines p-values and fold-changes with name harmonization: "Easy-Amanida is a new tool that combines two R packages, "amanida" and "webchem", to enable meta-analysis of aggregate statistical data"
- [abstract] Aggregate data types and harmonization rationale: "to enable meta-analysis of aggregate statistical data, like p-value and fold-change, while ensuring the compounds naming harmonization"
- [intro] Shiny implementation of the Amanida package: "Easy-Amanida is a web-based app, developed with Shiny, that implements the R package Amanida."
- [readme] Naming-combination workflow in the app: "The Easy-Amanida app is implemented in Shiny, an R package add-on for interactive web apps, and provides a workflow to optimize naming combination."
- [methods] Methods statement on the meta-analysis scope: "enable meta-analysis of aggregate statistical data, like p-value and fold-change, while ensuring the compounds naming harmonization"
