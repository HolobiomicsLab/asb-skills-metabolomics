---
name: metabolite-identifier-mapping
description: Use when a list of differential metabolites from LC-MS/MS untargeted
  or targeted metabolomics (e.g., from XCMS processing of >20,000 m/z values) needs
  functional pathway annotation, and you must first reconcile compound identifiers
  against HMDB before submitting to KEGG-based enrichment tools.
when_to_use_negative:
- When metabolite identifiers already exist as HMDB or KEGG compound IDs and the mapping
  step is redundant
- When pathway-level functional annotation is not required, and the goal is purely
  statistical batch effect correction (use dbnorm instead)
- When the metabolite list is small and unlikely to contain ≥2 metabolites per KEGG
  pathway, since the enrichment threshold (≥2 metabolites) would render results non-significant
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3390
- http://edamontology.org/topic_0154
tools:
- name: Human Metabolome Database (HMDB)
  role: Reference database for filtering and mapping detected metabolite features
    to standardized compound identifiers
- name: ConsensusPathDB
  role: Web tool for over-representation analysis against KEGG pathways (4289 compound
    IDs) using hypergeometric test and FDR correction
- name: KEGG pathways database
  role: Predefined pathway reference set used for metabolite set enrichment analysis
- name: XCMS
  role: Upstream LC-MS/MS data processing tool that produces the metabolite feature
    list (m/z values) subject to identifier mapping
- name: dbnorm
  role: Upstream batch-effect correction tool that generates the normalized metabolite
    intensity matrix used for differential analysis feeding into identifier mapping
  repo: https://github.com/NBDZ/dbnorm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:89dabdb47c4c2d4c58b8b9f0e599937e2c02a916c130065a812f7473b9e7d8e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ConsensusPathDB
  - Human Metabolome Database (HMDB)
  - KEGG pathways database
  - XCMS
  - dbnorm
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3390
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# metabolite-identifier-mapping

## Summary

Maps detected metabolite feature identifiers to standardized HMDB compound IDs and then performs KEGG-based metabolite set enrichment analysis (MSEA) in ConsensusPathDB, enabling functional annotation of dysregulated metabolites detected by LC-MS/MS targeted metabolomics.

## When to use

Use when a list of differential metabolites from LC-MS/MS untargeted or targeted metabolomics (e.g., from XCMS processing of >20,000 m/z values) needs functional pathway annotation, and you must first reconcile compound identifiers against HMDB before submitting to KEGG-based enrichment tools.

## When NOT to use

- When metabolite identifiers already exist as HMDB or KEGG compound IDs and the mapping step is redundant
- When pathway-level functional annotation is not required, and the goal is purely statistical batch effect correction (use dbnorm instead)
- When the metabolite list is small and unlikely to contain ≥2 metabolites per KEGG pathway, since the enrichment threshold (≥2 metabolites) would render results non-significant

## Inputs

- List of differential metabolites with Log2FC > 1.5 and BH-adjusted p-value < 0.05 from LC-MS/MS metabolomics (e.g., from XCMS processing)
- HMDB compound identifier mapping (via Human Metabolome Database, http://www.hmdb.ca)

## Outputs

- Filtered set of HMDB-confirmed metabolite identifiers
- Pathway enrichment table from ConsensusPathDB (KEGG pathways) with metabolite coverage, q-values, and negative log2-transformed q-values
- Significantly dysregulated KEGG pathways (≥2 metabolites per pathway, q-value < 0.05) e.g., alpha-Linolenic acid metabolism, fatty acid biosynthesis

## How to apply

First, load the list of differential metabolites (e.g., Log2FC > 1.5 and BH-adjusted p < 0.05) and filter it against the Human Metabolome Database (HMDB, http://www.hmdb.ca) to retain only confirmed metabolite hits with valid HMDB identifiers. Second, submit those HMDB-confirmed compound IDs to ConsensusPathDB (http://consensuspathdb.org/) for over-representation analysis against the predefined KEGG pathways database, which contains 4289 compound IDs. Third, compute pathway significance via hypergeometric test and correct p-values for multiple testing using false discovery rate (q-values). Fourth, retain significantly dysregulated pathways meeting the criteria of at least two metabolites per pathway and q-value < 0.05. The rationale is that HMDB filtering removes unannotated or ambiguous m/z features, ensuring that only biologically meaningful metabolites drive enrichment signals.

## Related tools

- **Human Metabolome Database (HMDB)** (Reference database for filtering and mapping detected metabolite features to standardized compound identifiers)
- **ConsensusPathDB** (Web tool for over-representation analysis against KEGG pathways (4289 compound IDs) using hypergeometric test and FDR correction)
- **KEGG pathways database** (Predefined pathway reference set used for metabolite set enrichment analysis)
- **XCMS** (Upstream LC-MS/MS data processing tool that produces the metabolite feature list (m/z values) subject to identifier mapping)
- **dbnorm** (Upstream batch-effect correction tool that generates the normalized metabolite intensity matrix used for differential analysis feeding into identifier mapping) — https://github.com/NBDZ/dbnorm

## Examples

```
ConsensusPathDB web interface (http://consensuspathdb.org/) — upload HMDB-confirmed metabolite IDs; e.g., submit list via `curl -d "compounds=HMDB0000042,HMDB0000176..." http://consensuspathdb.org/` (web tool, no local CLI); upstream R call: `R> library(dbnorm); R> df<-read.csv("mydata.csv",sep=",",header=T,row.names=1); R> Visodbnorm(df)`
```

## Evaluation signals

- The filtered metabolite list must contain only compounds that resolve to valid HMDB identifiers after querying http://www.hmdb.ca
- Enrichment output must include metabolite coverage (relative number of input metabolites defining each pathway relative to the whole pathway metabolite set) and negative log2-transformed q-values
- All reported pathways must have ≥2 metabolites and q-value < 0.05; pathways below these thresholds must be excluded
- Cross-check that enriched pathways reflect the biological phenotype — e.g., HFD-associated lipid metabolism pathways (alpha-Linolenic acid metabolism, fatty acid biosynthesis) are expected for HFD-dysregulated metabolite lists
- Confirm that FDR (q-value) correction was applied to raw hypergeometric p-values rather than reporting uncorrected p-values

## Limitations

- Enrichment requires at least 2 metabolites per pathway to be considered significant (q-value < 0.05), so pathways represented by only one metabolite will be missed
- Only metabolites matching the HMDB database are retained — unmapped m/z features are excluded, potentially discarding biologically relevant but unannotated metabolites
- The KEGG pathways database referenced by ConsensusPathDB contains a fixed set of 4289 compound IDs, limiting coverage of novel or exotic metabolites
- Enrichment depends on the upstream batch-effect correction quality; poor normalization (e.g., residual batch variation > 1%) can distort the differential metabolite list and hence the enrichment result

## Evidence

- [methods] Filter the significant list against HMDB to retain only potential metabolite hits: "searched and filtered against a Human Metabolome Database (HMDB; http://www.hmdb.ca)"
- [methods] Submit HMDB-confirmed metabolite identifiers to ConsensusPathDB for KEGG pathway over-representation analysis: "The list of identifiers was mapped to predefined KEGG pathways database enlisted by 4289 compound IDs."
- [results] Compute pathway significance with hypergeometric test and correct p-values with FDR (q-values): "we next annotated the metabolites that were significantly influenced by HFD to functional pathways by using metabolite set enrichment analysis (MSEA)"
- [results] Enriched pathways must include lipid metabolism categories, specifically alpha-Linolenic acid metabolism and fatty acid biosynthesis: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [abstract] The tool dbnorm handles upstream normalization and generates the differential metabolite list used for enrichment: [evidence span withheld — anchor status: not_found]
- [readme] The README documents dbnorm installation and function calls, which feed into the identifier mapping workflow: "*dbnorm* includes 11 distinct functions for pre-processing of data and estimation of missing values, conventional functions for batch effect correction based on statistical models, as well as"
