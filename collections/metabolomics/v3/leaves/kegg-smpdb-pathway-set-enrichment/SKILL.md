---
name: kegg-smpdb-pathway-set-enrichment
description: Use when you have sex-stratified (or group-stratified) differential metabolite
  tables and need to identify which biological pathways are enriched in one group
  versus another, mirroring the ferroptosis/bile-acid (male) and glutathione/cAMP/cGMP-PKG
  (female) CRC results.
when_to_use_negative:
- You need full end-to-end LC-MS raw data processing (peak picking, annotation) rather
  than downstream pathway enrichment — metPath operates on already-identified differential
  metabolites
- Input is raw mzML/LC-MS data rather than a processed differential-metabolite list
  with metabolite identifiers
- You require pathway analysis tools not wrapped by tidyMass — the article notes 'some
  widely used metabolomics tools are not wrapped up or supported in tidyMass', though
  mass_dataset conversion may bridge this
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metpath
  role: Performs KEGG and SMPDB over-representation enrichment (Hypergeometric/Fisher's
    exact) on differential metabolite query sets
  repo: https://gitlab.com/tidymass/tidymass
- name: tidyMass
  role: Modular R ecosystem providing the uniform mass_dataset data structure that
    metPath and massstat share
  repo: https://github.com/tidymass/tidymass
- name: massstat
  role: Upstream statistical analysis step producing the differential-metabolite foreground
    lists per sex
  repo: https://www.tidymass.org/
- name: massdataset
  role: Dependency of metPath providing the uniform data format for processed metabolomics
    data
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: remotes
  role: Used to install metPath and tidymass from GitLab/GitHub
  repo: https://gitlab.com/tidymass/tidymass
- name: ggplot2
  role: Renders the per-sex pathway-enrichment summary figure analogous to Fig. 4b
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:b737bf242eb2711a360dfdf9b41d1b9b79a7f37afcef79c2868343e708d353ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dplyr
  - ggplot2
  - massdataset
  - massstat
  - metpath
  - remotes
  - tidyMass
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# kegg-smpdb-pathway-set-enrichment

## Summary

Run over-representation pathway enrichment (KEGG and SMPDB) on sex-stratified differential metabolite lists from colorectal cancer tumors using metPath from the tidyMass ecosystem, producing male-only vs female-only enriched pathway sets.

## When to use

Use when you have sex-stratified (or group-stratified) differential metabolite tables and need to identify which biological pathways are enriched in one group versus another, mirroring the ferroptosis/bile-acid (male) and glutathione/cAMP/cGMP-PKG (female) CRC results.

## When NOT to use

- You need full end-to-end LC-MS raw data processing (peak picking, annotation) rather than downstream pathway enrichment — metPath operates on already-identified differential metabolites
- Input is raw mzML/LC-MS data rather than a processed differential-metabolite list with metabolite identifiers
- You require pathway analysis tools not wrapped by tidyMass — the article notes 'some widely used metabolomics tools are not wrapped up or supported in tidyMass', though mass_dataset conversion may bridge this

## Inputs

- Male-tumor differential-metabolite table from massstat differential analysis (task_004)
- Female-tumor differential-metabolite table from massstat differential analysis (task_004)
- KEGG pathway metabolite-set database (universe background)
- SMPDB pathway metabolite-set database (universe background)

## Outputs

- enrichment_results.csv with columns: pathway id, pathway name, database (KEGG or SMPDB), p value, adjusted p value, and sex(es) enriched
- fig4b_pathway_enrichment.png — ggplot2 figure showing male-enriched vs female-enriched pathways analogous to Fig. 4b

## How to apply

Load the male-tumor and female-tumor differential-metabolite tables from the massstat differential-analysis step as the foreground query sets for each sex. Install metPath from the tidyMass ecosystem (remotes::install_gitlab("tidymass/metpath")) together with its massdataset dependency, preserving tidyMass's modular package structure. For each sex, run metPath::enrich_kegg() and metPath::enrich_smpdb() using Hypergeometric/Fisher's exact over-representation testing, passing the differential metabolites as the query set and the KEGG/SMPDB metabolite sets as the universe background. Collect enriched pathways per sex, annotate each as male-only, female-only, or both, and retain pathways with adjusted p value below the enrichment cutoff shown in Fig. 4b. Assemble results into enrichment_results.csv (pathway id, name, database, p value, adjusted p value, sex) and render a Fig. 4b-analogous figure with ggplot2 contrasting male-enriched (ferroptosis, bile-acid synthesis) against female-enriched (glutathione metabolism, cAMP signaling, cGMP-PKG signaling) pathways.

## Related tools

- **metpath** (Performs KEGG and SMPDB over-representation enrichment (Hypergeometric/Fisher's exact) on differential metabolite query sets) — https://gitlab.com/tidymass/tidymass
- **tidyMass** (Modular R ecosystem providing the uniform mass_dataset data structure that metPath and massstat share) — https://github.com/tidymass/tidymass
- **massstat** (Upstream statistical analysis step producing the differential-metabolite foreground lists per sex) — https://www.tidymass.org/
- **massdataset** (Dependency of metPath providing the uniform data format for processed metabolomics data) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **remotes** (Used to install metPath and tidymass from GitLab/GitHub) — https://gitlab.com/tidymass/tidymass
- **ggplot2** (Renders the per-sex pathway-enrichment summary figure analogous to Fig. 4b)

## Examples

```
remotes::install_gitlab("tidymass/metpath"); res_male <- metpath::enrich_kegg(diff_metabolites_male, universe = all_metabolites)
```

## Evaluation signals

- Ferroptosis and bile-acid synthesis appear exclusively among male-tumor enriched pathways, matching the reported Fig. 4b pattern
- Glutathione metabolism, cAMP signaling, and cGMP-PKG signaling appear exclusively among female-tumor enriched pathways and absent from males
- enrichment_results.csv contains all required columns (pathway id, name, database, p value, adjusted p value, sex) with database correctly labeled KEGG or SMPDB
- Only pathways with adjusted p value below the enrichment cutoff shown in Fig. 4b are retained
- Enrichment uses the full KEGG/SMPDB metabolite sets as universe background, not the query list itself (Fisher's exact setup is valid)

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though mass_dataset can be converted to other data structures
- Significance is judged against the enrichment cutoff shown in Fig. 4b, so exact thresholds must be read from the source figure rather than a universal default
- Results depend on the quality of the upstream massstat differential-metabolite tables and level of metabolite annotation

## Evidence

- [abstract] a comprehensive computational framework that can achieve the shareable and reproducible workflow needs of data processing and analysis for LC-MS-based untargeted metabolomics: "a comprehensive computational framework that can achieve the shareable and reproducible workflow needs of data processing and analysis for LC-MS-based untargeted metabolomics"
- [intro] tidyMass is an object-oriented workflow with a uniform data structure mass_dataset that is the kernel of tidyMass: "the tidyMass is an object-oriented workflow"
- [intro] tidyMass has high interoperability with other existing tools, converting mass_dataset class with other data structures required by other tools: [evidence span withheld — anchor status: not_found]
- [readme] You can install tidymass from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [discussion] Some widely used metabolomics tools are not wrapped up or supported in tidyMass: "some widely used metabolomics tools are not wrapped up or supported in tidyMass"
