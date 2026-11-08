---
name: go-term-enrichment-interpretation
description: Use when a time-course or multi-group gene expression dataset has been
  clustered (e.g., fuzzy c-means on KO abundance) and you need the GO terms significantly
  enriched for each cluster via pattern-specified GRSA.
when_to_use_negative:
- The abundance table has been prefiltered to a short list of significant features
  — GRSA is threshold-free and needs background information retained
- The design is a simple two-group comparison with no cluster or longitudinal structure,
  where a standard single-group enrichment method suffices
- Features are not mappable to KEGG K numbers or gene symbols recognized by the pathway/GO
  databases (e.g., unannotated compounds without C numbers)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
tools:
- name: ReporterScore
  role: R package implementing GRSA, fuzzy c-means clustering helpers (c_means/RSA_by_cm,
    extract_cluster, GRSA_by_cm), and GO/pathway databases used to obtain per-cluster
    GO enrichments
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed before ReporterScore for the GitHub development installation
  repo: github.com/Asa12138/pcutils
- name: devtools
  role: Used to install ReporterScore and pcutils from GitHub
- name: R
  role: Runtime environment in which the enrichment workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:2394bbc4acbb2f0bdad60d52910c48b258f6beb5f5fd337f015fcec49f7e9c64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# go-term-enrichment-interpretation

## Summary

Interprets gene-set enrichment results at the GO-term level using the ReporterScore package's GRSA framework, e.g., identifying 'Heart process (GO:0003015)' as enriched in the day-30 cardiomyocyte expression cluster of the Liu et al. 2017 differentiation transcriptome.

## When to use

Use when a time-course or multi-group gene expression dataset has been clustered (e.g., fuzzy c-means on KO abundance) and you need the GO terms significantly enriched for each cluster via pattern-specified GRSA.

## When NOT to use

- The abundance table has been prefiltered to a short list of significant features — GRSA is threshold-free and needs background information retained
- The design is a simple two-group comparison with no cluster or longitudinal structure, where a standard single-group enrichment method suffices
- Features are not mappable to KEGG K numbers or gene symbols recognized by the pathway/GO databases (e.g., unannotated compounds without C numbers)

## Inputs

- Gene-by-sample expression/time-course abundance table (rownames = gene symbols or KO IDs, colnames = samples; counts or TPM/FPKM/RPKM)
- Sample metadata table (rownames matching abundance-table colnames, grouping or time-point columns)
- Organism pathway/GO database (e.g., load_org_pathway(), load_GOlist())
- Number of fuzzy c-means clusters (k_num) and variance filter setting (filter_var)

## Outputs

- Per-cluster enriched GO/pathway term table with significance (e.g., 'Heart process (GO:0003015)' for the day-30 cardiomyocyte Cluster 6)
- Compiled enrichment table saved as cluster_go_enrichment.csv
- rsa_cm_res object with per-cluster gene assignments from extract_cluster()

## How to apply

Load the gene expression table and sample metadata in R with the ReporterScore package (installed via devtools::install_github("Asa12138/ReporterScore") after pcutils). Map genes to KEGG Orthology with gene2ko() using the organism pathway database from load_org_pathway(), then fit fuzzy c-means clusters on the KO abundance table with c_means()/RSA_by_cm(), specifying k_num and filter_var to drop low-variance features (the input abundance table should not be prefiltered, since GRSA is threshold-free). Extract individual cluster assignments from the rsa_cm_res object with extract_cluster(), then run pattern-specified GRSA per cluster with GRSA_by_cm() to obtain enriched GO/pathway terms. Compile the enriched terms per cluster into a table (e.g., cluster_go_enrichment.csv) and check that biologically coherent terms—such as 'Heart process (GO:0003015)' for a cluster expressed only in the differentiated cardiomyocyte stage (day 30)—appear, which validates the enrichment.

## Related tools

- **ReporterScore** (R package implementing GRSA, fuzzy c-means clustering helpers (c_means/RSA_by_cm, extract_cluster, GRSA_by_cm), and GO/pathway databases used to obtain per-cluster GO enrichments) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed before ReporterScore for the GitHub development installation) — github.com/Asa12138/pcutils
- **devtools** (Used to install ReporterScore and pcutils from GitHub)
- **R** (Runtime environment in which the enrichment workflow is executed)

## Examples

```
devtools::install_github("Asa12138/pcutils"); devtools::install_github("Asa12138/ReporterScore"); library(ReporterScore); rsa_cm_res <- RSA_by_cm(ko_df, k_num=6); res <- GRSA_by_cm(rsa_cm_res, type="GO")
```

## Evaluation signals

- 'Heart process (GO:0003015)' appears among Cluster 6 enrichments for the day-30 cardiomyocyte stage, matching the known biology of differentiated cardiomyocytes
- cluster_go_enrichment.csv exists and contains enriched terms for every extracted cluster
- Many genes match Cluster 6's expression pattern, confirming the fuzzy c-means assignment is non-trivial
- Metadata rownames match abundance-table colnames (all(rownames(metadata) %in% colnames(KO_abundance)) is TRUE)
- The input abundance table was not prefiltered, preserving threshold-free GRSA behavior

## Limitations

- No version number is stated for the ReporterScore package in the article, limiting reproducibility
- GRSA exhibits slightly lower specificity than some methods, so enriched GO terms require additional experimental validation
- There is no established gold standard for comparing enrichment methods, and specificity cannot be accurately evaluated because true negatives are hard to define due to incomplete databases
- Pathway databases should be updated (update_KO_file(), update_GOlist()) since using the latest database is important and built-in KEGG data may be outdated

## Evidence

- [results] GRSA on the cardiomyocyte differentiation clusters identified 'Heart process (GO:0003015)' as a significantly enriched GO term for Cluster 6: [evidence span withheld — anchor status: not_found]
- [intro] the GRSA supports multi-group and longitudinal experimental designs: "Importantly, the GRSA supports multi-group and longitudinal experimental designs because of the included multi-group-compatible statistical methods"
- [methods] devtools::install_github("Asa12138/ReporterScore"): "devtools::install_github("Asa12138/ReporterScore")
library(ReporterScore)"
- [readme] the input abundance table should not be prefiltered: [evidence span withheld — anchor status: external_doc]
- [discussion] GRSA considers all KOs involved in the pathway compared to hypergeometric tests: "GRSA considers all KOs involved in the pathway compared to hypergeometric tests that only consider a pre-defined list"
