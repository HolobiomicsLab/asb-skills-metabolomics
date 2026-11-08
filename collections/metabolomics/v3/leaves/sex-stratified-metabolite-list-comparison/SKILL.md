---
name: sex-stratified-metabolite-list-comparison
description: Use when sex-specific differential-metabolite tables exist (from a massstat
  differential-analysis step) and the goal is to determine which KEGG/SMPDB pathways
  are enriched only in male tumors versus only in female tumors, as in Fig. 4b.
when_to_use_negative:
- Raw LC-MS data still requires peak picking, cleaning, or annotation — run the tidyMass
  raw-data processing and massstat differential-analysis steps first, since this skill
  consumes their output tables
- The cohort is not stratified by patient sex (no male/female split of differential
  metabolites is available)
- The goal is metabolite annotation or identification rather than over-representation
  of already-annotated differential metabolites in pathways
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metpath
  role: Runs enrich_kegg() and enrich_smpdb() Hypergeometric/Fisher's exact over-representation
    tests on the per-sex differential-metabolite query sets
- name: tidyMass
  role: Modular R ecosystem providing the massdataset data structure and installation
    entry point for metpath and massstat
  repo: https://github.com/tidymass/tidymass
- name: ggplot2
  role: Renders the Fig. 4b-analogous summary figure contrasting male-enriched and
    female-enriched pathways
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:77be6f24d3f43c1dc21eb41ef986439a4d6bc5c3083f1485bf86a244e5bd395c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - dplyr
  - ggplot2
  - massstat
  - metpath
  - remotes
  - tidyMass
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sex-stratified-metabolite-list-comparison

## Summary

Sex-stratified pathway-enrichment comparison: take the male-tumor and female-tumor differential-metabolite tables from the upstream massstat differential-analysis step and run metPath KEGG/SMPDB over-representation testing per sex to identify pathways enriched exclusively in male versus female colorectal-cancer tumors (ferroptosis/bile acid in males; glutathione/cAMP/cGMP-PKG in females).

## When to use

Use when sex-specific differential-metabolite tables exist (from a massstat differential-analysis step) and the goal is to determine which KEGG/SMPDB pathways are enriched only in male tumors versus only in female tumors, as in Fig. 4b.

## When NOT to use

- Raw LC-MS data still requires peak picking, cleaning, or annotation — run the tidyMass raw-data processing and massstat differential-analysis steps first, since this skill consumes their output tables
- The cohort is not stratified by patient sex (no male/female split of differential metabolites is available)
- The goal is metabolite annotation or identification rather than over-representation of already-annotated differential metabolites in pathways

## Inputs

- male-tumor differential-metabolite table from the massstat differential-analysis step
- female-tumor differential-metabolite table from the massstat differential-analysis step
- KEGG pathway metabolite-set database
- SMPDB pathway metabolite-set database

## Outputs

- enrichment_results.csv (pathway id, pathway name, database [KEGG or SMPDB], p value, adjusted p value, sex(es) enriched)
- fig4b_pathway_enrichment.png (ggplot2 summary figure analogous to Fig. 4b)
- male-only and female-only enriched-pathway sets

## How to apply

Load the male-tumor and female-tumor differential-metabolite tables produced by the massstat step as the per-sex foreground metabolite lists. Attach metPath from the tidyMass ecosystem (remotes::install_gitlab("tidymass/metpath")) together with its massdataset dependency, matching tidyMass's modular package architecture. For each sex, run metPath::enrich_kegg() and metPath::enrich_smpdb() using Hypergeometric/Fisher's exact over-representation testing, with the differential metabolites as the query set and the KEGG/SMPDB metabolite sets as the universe background. Retain pathways with adjusted p value below the enrichment cutoff shown in Fig. 4b, then annotate each pathway as male-only, female-only, or both. Assemble the results into enrichment_results.csv (pathway id, name, database, p value, adjusted p value, sex(es)) and render a fig4b_pathway_enrichment.png analog with ggplot2 comparing male-enriched (ferroptosis, bile-acid synthesis) versus female-enriched (glutathione metabolism, cAMP signaling, cGMP-PKG signaling) pathways.

## Related tools

- **metpath** (Runs enrich_kegg() and enrich_smpdb() Hypergeometric/Fisher's exact over-representation tests on the per-sex differential-metabolite query sets)
- **tidyMass** (Modular R ecosystem providing the massdataset data structure and installation entry point for metpath and massstat) — https://github.com/tidymass/tidymass
- **ggplot2** (Renders the Fig. 4b-analogous summary figure contrasting male-enriched and female-enriched pathways)

## Examples

```
remotes::install_gitlab("tidymass/metpath")
```

## Evaluation signals

- Ferroptosis and bile-acid synthesis appear exclusively among male-tumor enriched pathways
- Glutathione metabolism, cAMP signaling, and cGMP-PKG signaling appear exclusively among female-tumor enriched pathways
- Male-only and female-only pathway sets match the reported Fig. 4b enrichment pattern
- All retained pathways have adjusted p value below the enrichment cutoff shown in Fig. 4b
- enrichment_results.csv contains the required columns: pathway id, pathway name, database (KEGG or SMPDB), p value, adjusted p value, and sex(es)

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, so interoperability relies on converting the mass_dataset class to other tools' data structures
- Over-rerepresentation testing is limited to the pathway coverage of KEGG and SMPDB metabolite sets
- Downstream analysis quality depends on upstream annotation filtering (only level-2 annotations were used, redundant annotations removed by annotation score)

## Evidence

- [abstract] abstract: "a comprehensive computational framework that can achieve the shareable and reproducible workflow needs of data processing and analysis for LC-MS-based untargeted metabolomics"
- [intro] intro: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis, and biological function"
- [discussion] discussion: "some widely used metabolomics tools are not wrapped up or supported in tidyMass"
- [readme] readme: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
