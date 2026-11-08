---
name: metabolite-pathway-over-representation-analysis
description: Use when you have per-group differential metabolite tables (e.g., male
  vs female tumors from a massstat differential analysis) and need to know which KEGG/SMPDB
  biological pathways are specifically enriched in one group but not the other.
when_to_use_negative:
- Input is raw LC-MS data still needing peak picking, data cleaning, or metabolite
  annotation — pathway enrichment must be preceded by the tidyMass data-processing
  and differential-analysis steps that produce the differential-metabolite lists
- Metabolites lack pathway-compatible identifiers (annotations below level 2 or unmapped
  to KEGG/SMPDB sets), so the query cannot be matched to the universe background
- The goal is quantitative pathway/topology analysis or flux modeling rather than
  categorical over-representation of a differential hit list
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0092
tools:
- name: metpath
  role: 'Performs the pathway over-representation testing: enrich_kegg() and enrich_smpdb()
    run Hypergeometric/Fisher''s exact tests with the differential metabolites as
    query against KEGG/SMPDB metabolite sets as universe'
- name: massdataset
  role: Dependency of metPath providing the uniform mass_dataset data structure that
    the metabolite lists flow through, enabling traceability of prior analysis steps
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: massstat
  role: Upstream step that produces the sex-stratified differential-metabolite tables
    used as the enrichment query sets
- name: tidymass
  role: Umbrella package organizing the tidyMass ecosystem (metpath, massdataset,
    massstat) installed via remotes from GitLab or GitHub
  repo: https://github.com/tidymass/tidymass
- name: ggplot2
  role: Renders the combined per-sex enrichment summary figure analogous to Fig. 4b
- name: remotes
  role: Installs metPath from the tidyMass GitLab repository and its dependencies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:8b9c15021ad0479d2a5e663dafd6d99f7371809665321876e75a3e5a298271c5
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
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metabolite-pathway-over-representation-analysis

## Summary

Runs over-representation (Hypergeometric/Fisher's exact) pathway enrichment on sex-stratified differential-metabolite lists from colorectal cancer tumors using metPath (tidyMass ecosystem) against KEGG and SMPDB metabolite sets, then partitions enriched pathways into male-only, female-only, or shared sets.

## When to use

Use when you have per-group differential metabolite tables (e.g., male vs female tumors from a massstat differential analysis) and need to know which KEGG/SMPDB biological pathways are specifically enriched in one group but not the other.

## When NOT to use

- Input is raw LC-MS data still needing peak picking, data cleaning, or metabolite annotation — pathway enrichment must be preceded by the tidyMass data-processing and differential-analysis steps that produce the differential-metabolite lists
- Metabolites lack pathway-compatible identifiers (annotations below level 2 or unmapped to KEGG/SMPDB sets), so the query cannot be matched to the universe background
- The goal is quantitative pathway/topology analysis or flux modeling rather than categorical over-representation of a differential hit list

## Inputs

- Per-group differential-metabolite tables (e.g., male-tumor and female-tumor DE metabolite lists from the massstat differential-analysis step, task_004)
- KEGG and SMPDB pathway metabolite sets as the universe background (via metPath)
- Adjusted-p significance cutoff as shown in the reference figure (Fig. 4b)

## Outputs

- enrichment_results.csv with columns: pathway id, pathway name, database (KEGG or SMPDB), p value, adjusted p value, and sex(es) in which the pathway is enriched
- Summary enrichment figure analogous to Fig. 4b (fig4b_pathway_enrichment.png) rendered with ggplot2
- Sex-specific pathway classification: male-only, female-only, or shared enriched pathway sets

## How to apply

Load the differential-metabolite tables produced by the upstream massstat differential-analysis step as the foreground query set for each group (here, male-tumor and female-tumor metabolites). Install/attach metPath from the tidyMass ecosystem (remotes::install_gitlab("tidymass/metpath")) together with its dependency massdataset, matching tidyMass's modular package architecture. For each group run metPath::enrich_kegg() and metPath::enrich_smpdb(), using Hypergeometric/Fisher's exact over-representation testing with the differential metabolites as the query and the KEGG/SMPDB metabolite sets as the universe background. Apply the paper's significance criterion — pathways with adjusted p value below the enrichment cutoff shown in Fig. 4b — and annotate each pathway as male-only, female-only, or enriched in both. Rationale: pathway over-representation is the biological-function-mining stage of the LC-MS untargeted metabolomics workflow that tidyMass frames, and metPath interoperates with mass_dataset objects so enriched sets can be traced back through the pipeline. Assemble results into enrichment_results.csv and render a Fig. 4b-style summary figure with ggplot2 (ferroptosis and bile acid synthesis for males vs glutathione metabolism, cAMP and cGMP-PKG signaling for females).

## Related tools

- **metpath** (Performs the pathway over-representation testing: enrich_kegg() and enrich_smpdb() run Hypergeometric/Fisher's exact tests with the differential metabolites as query against KEGG/SMPDB metabolite sets as universe)
- **massdataset** (Dependency of metPath providing the uniform mass_dataset data structure that the metabolite lists flow through, enabling traceability of prior analysis steps) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **massstat** (Upstream step that produces the sex-stratified differential-metabolite tables used as the enrichment query sets)
- **tidymass** (Umbrella package organizing the tidyMass ecosystem (metpath, massdataset, massstat) installed via remotes from GitLab or GitHub) — https://github.com/tidymass/tidymass
- **ggplot2** (Renders the combined per-sex enrichment summary figure analogous to Fig. 4b)
- **remotes** (Installs metPath from the tidyMass GitLab repository and its dependencies)

## Examples

```
remotes::install_gitlab("tidymass/metpath"); remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- Male-only enriched pathway set contains exactly ferroptosis and bile acid synthesis; female-only set contains exactly glutathione metabolism, cAMP signaling, and cGMP-PKG signaling — i.e., the male-only and female-only sets match the reported Fig. 4b enrichment pattern
- enrichment_results.csv contains the required schema: pathway id, pathway name, database (KEGG or SMPDB), p value, adjusted p value, and sex(es) enriched
- Every retained pathway has an adjusted p value below the enrichment cutoff shown in Fig. 4b
- No pathway classified as male-only also appears in the female-only set (and vice versa), and shared pathways are flagged as enriched in both sexes
- Each enrichment run used the full KEGG/SMPDB metabolite-set universe as background, not the detected feature list alone

## Limitations

- The paper notes that 'some widely used metabolomics tools are not wrapped up or supported in tidyMass' — metPath must be installed separately via remotes::install_gitlab("tidymass/metpath") and interoperability depends on the mass_dataset data structure
- Enrichment quality depends entirely on the upstream differential analysis and annotation depth; the tidyMass workflow used only level-2 annotations and removed redundant annotations by annotation score, which constrains which metabolites can map to pathway sets
- Over-representation treats metabolites as an unordered hit list and cannot rank pathway perturbation strength or use metabolite fold-changes

## Evidence

- [abstract] modular architecture supports metPath integration: "The modular architecture makes tidyMass a highly flexible and extensible tool, so other users can improve it and integrate it with their own pipeline easily"
- [intro] biological function mining is the final workflow stage: "requiring the optimization of multiple steps such as raw data processing, data cleaning, data quality control and assessment, metabolite annotation, statistical analysis, and biological function"
- [intro] tidyMass workflow framing for metabolomics analysis: "A typical full workflow for LC-MS-based untargeted metabolomics includes sample collection, data acquisition, data analysis, and biological interpretation"
- [intro] mass_dataset interoperability with metPath: [evidence span withheld — anchor status: not_found]
- [intro] level-2 annotation restriction constrains pathway mapping: "Only the annotations with level 2 were used for subsequent analysis"
- [readme] installing tidyMass ecosystem packages via remotes: "remotes::install_gitlab("tidymass/tidymass")"
- [readme] tidymass as the organizing umbrella package: [evidence span withheld — anchor status: not_found]
