---
name: pathway-enrichment-results-visualization
description: Use when you have sex-specific (or group-specific) differential metabolite
  tables from a massstat differential-analysis step and need to identify which KEGG/SMPDB
  pathways are enriched in one group but not another.
when_to_use_negative:
- Input is raw LC-MS data or a peak/feature table that has not yet passed differential
  analysis — use the tidyMass raw-data-processing and statistical-analysis packages
  first.
- You need pathway mapping from annotated (level-2) metabolite IDs but only have unannotated
  features — metabolite annotation must precede biological function mining.
- You want pathway analysis of transcriptomic or proteomic data rather than metabolite
  sets, since the metPath query sets here are metabolite-level KEGG/SMPDB sets.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: metpath
  role: Performs enrich_kegg() and enrich_smpdb() Hypergeometric/Fisher's exact over-representation
    testing on the differential-metabolite query sets
  repo: https://gitlab.com/tidymass/tidymass
- name: massdataset
  role: Dependency of metPath providing the uniform mass_dataset data structure used
    to carry differential metabolite lists
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: ggplot2
  role: Renders the Fig. 4b-style pathway-enrichment summary figure comparing male-
    and female-enriched pathways
- name: tidymass
  role: Umbrella package organizing the tidyMass ecosystem from which metPath is installed
    via remotes
  repo: https://gitlab.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:7688d778d9b5106877f44b0fb47919b10bb9f31d3d879e902bad326ef93d8e11
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
  - tidymass
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pathway-enrichment-results-visualization

## Summary

Run over-representation pathway enrichment (metPath) on sex-stratified differential metabolite lists from LC-MS colorectal-cancer data and render a per-sex summary figure. It implements the tidyMass 'biological function mining' step, converting differential-metabolite tables into KEGG/SMPDB pathway-level findings.

## When to use

Use when you have sex-specific (or group-specific) differential metabolite tables from a massstat differential-analysis step and need to identify which KEGG/SMPDB pathways are enriched in one group but not another.

## When NOT to use

- Input is raw LC-MS data or a peak/feature table that has not yet passed differential analysis — use the tidyMass raw-data-processing and statistical-analysis packages first.
- You need pathway mapping from annotated (level-2) metabolite IDs but only have unannotated features — metabolite annotation must precede biological function mining.
- You want pathway analysis of transcriptomic or proteomic data rather than metabolite sets, since the metPath query sets here are metabolite-level KEGG/SMPDB sets.

## Inputs

- Sex-stratified differential-metabolite tables from massstat differential analysis (male tumors, female tumors)
- KEGG metabolite pathway sets (via metPath)
- SMPDB metabolite pathway sets (via metPath)
- Universe/background metabolite list (full annotated metabolite set)

## Outputs

- enrichment_results.csv (pathway id, pathway name, database KEGG/SMPDB, p value, adjusted p value, sex(es) enriched)
- fig4b_pathway_enrichment.png (ggplot2 figure contrasting male-only vs female-only enriched pathways)

## How to apply

Load the male-tumor and female-tumor differential-metabolite tables from the upstream massstat differential-analysis step (task_004) as the foreground metabolite lists, and use the full annotated metabolite set as the universe background. Install metPath from the tidyMass ecosystem with remotes::install_gitlab("tidymass/metpath") together with its massdataset dependency, matching tidyMass's modular package architecture. For each sex, run metPath::enrich_kegg() and metPath::enrich_smpdb() using Hypergeometric/Fisher's exact over-representation testing with the differential metabolites as the query set. Classify each enriched pathway as male-only, female-only, or both, keeping pathways whose adjusted p value falls below the enrichment cutoff used in Fig. 4b of the paper. Assemble enrichment_results.csv (pathway id, name, database, p value, adjusted p value, sex(es)) and render a ggplot2 figure analogous to Fig. 4b contrasting male-enriched pathways (ferroptosis, bile-acid synthesis) with female-enriched ones (glutathione metabolism, cAMP signaling, cGMP-PKG signaling).

## Related tools

- **metpath** (Performs enrich_kegg() and enrich_smpdb() Hypergeometric/Fisher's exact over-representation testing on the differential-metabolite query sets) — https://gitlab.com/tidymass/tidymass
- **massdataset** (Dependency of metPath providing the uniform mass_dataset data structure used to carry differential metabolite lists) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **ggplot2** (Renders the Fig. 4b-style pathway-enrichment summary figure comparing male- and female-enriched pathways)
- **tidymass** (Umbrella package organizing the tidyMass ecosystem from which metPath is installed via remotes) — https://gitlab.com/tidymass/tidymass

## Examples

```
remotes::install_gitlab("tidymass/metpath"); metPath::enrich_kegg(diff_metabolites, universe = all_metabolites)
```

## Evaluation signals

- Ferroptosis and bile-acid synthesis appear exclusively among male-tumor enriched pathways, matching the reported Fig. 4b pattern.
- Glutathione metabolism, cAMP signaling, and cGMP-PKG signaling appear exclusively among female-tumor enriched pathways and not among males.
- enrichment_results.csv contains the required columns (pathway id, pathway name, database, p value, adjusted p value, sex) and every retained pathway has adjusted p value below the enrichment cutoff used in Fig. 4b.
- The figure shows a male-vs-female pathway partition consistent with the CSV, with database (KEGG or SMPDB) annotated per pathway.
- Enrichment uses differential metabolites as query and the full KEGG/SMPDB metabolite sets as universe background, so the background size exceeds the query set for each test.

## Limitations

- Only level-2 metabolite annotations were carried into downstream analysis in the tidyMass case study, so pathway coverage is limited to annotated metabolites.
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though mass_dataset can be converted to other data structures.
- Significance depends on the enrichment cutoff shown in Fig. 4b; different adjusted-p cutoffs will change the male-only/female-only partition.

## Evidence

- [results] Pathway enrichment showing sex-specific biology: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis, and biological function"
- [abstract] Modular design rationale for using metPath within tidyMass: "The modular architecture makes tidyMass a highly flexible and extensible tool, so other users can improve it and integrate it with their own pipeline easily"
- [intro] mass_dataset interoperability for carrying differential metabolite lists: [evidence span withheld — anchor status: not_found]
- [readme] Installing tidyMass-ecosystem packages via remotes: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
