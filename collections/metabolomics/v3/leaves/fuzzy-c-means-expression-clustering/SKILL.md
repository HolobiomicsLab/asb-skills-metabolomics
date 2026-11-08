---
name: fuzzy-c-means-expression-clustering
description: Use when a multi-time-point gene or KO abundance table needs to be partitioned
  into expression-pattern clusters (e.g., cardiomyocyte differentiation stages day
  0–30) so that pattern-specified GRSA can identify enriched GO/KEGG terms per cluster.
when_to_use_negative:
- The design is a simple two-group comparison with no time-course/pattern structure
  — plain reporter_score() or standard hypergeometric enrichment suffices.
- The abundance table has been prefiltred to only background/differential features
  — GRSA (and hence cluster-pattern enrichment) is threshold-free and requires the
  full background table.
- The input is raw reads rather than an annotated KO/gene/compound abundance table
  — features must first be mapped to K numbers, gene symbols, or C numbers.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ReporterScore (R package)
  role: Provides c_means()/RSA_by_cm(), extract_cluster(), and GRSA_by_cm() for fuzzy
    c-means clustering and cluster-pattern enrichment
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Dependency installed prior to ReporterScore when using the GitHub development
    version
  repo: github.com/Asa12138/pcutils
- name: devtools
  role: Used to install ReporterScore and pcutils from GitHub
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:44bc2d8f7abc8502208df9267f41c5972ac1ec973826ae509dd79cb461150917
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - ReporterScore (R package)
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# fuzzy-c-means-expression-clustering

## Summary

Clusters features (genes/KOs) across a time-course omics dataset using fuzzy c-means (via c_means()/RSA_by_cm() in the ReporterScore R package) to define expression patterns, which are then fed into pattern-specified GRSA for cluster-level pathway/GO enrichment.

## When to use

Use when a multi-time-point gene or KO abundance table needs to be partitioned into expression-pattern clusters (e.g., cardiomyocyte differentiation stages day 0–30) so that pattern-specified GRSA can identify enriched GO/KEGG terms per cluster.

## When NOT to use

- The design is a simple two-group comparison with no time-course/pattern structure — plain reporter_score() or standard hypergeometric enrichment suffices.
- The abundance table has been prefiltred to only background/differential features — GRSA (and hence cluster-pattern enrichment) is threshold-free and requires the full background table.
- The input is raw reads rather than an annotated KO/gene/compound abundance table — features must first be mapped to K numbers, gene symbols, or C numbers.

## Inputs

- Time-course gene expression table (rownames = gene symbols or K numbers, colnames = samples)
- Sample metadata table (rownames = samples, columns = groups/time points)
- Organism pathway database from load_org_pathway() / built-in KEGG and GO databases
- k_num (number of fuzzy c-means clusters) and filter_var (variance filter cutoff) parameters

## Outputs

- rsa_cm_res object with fuzzy c-means cluster fits and per-cluster patterns
- Per-feature cluster assignments from extract_cluster()
- Cluster-specific GO/KEGG enrichment results from GRSA_by_cm() (e.g., 'Heart process GO:0003015' for Cluster 6)
- Compiled enrichment table (cluster_go_enrichment.csv)

## How to apply

Load the time-course abundance table and sample metadata in R after installing ReporterScore via devtools (with the pcutils dependency). Map genes to KEGG Orthology entries with gene2ko() using the organism pathway database from load_org_pathway() so clustering operates on KO abundances. Fit fuzzy c-means clusters on the KO abundance table with c_means()/RSA_by_cm(), choosing the number of clusters k_num and setting filter_var to drop low-variance features before clustering. Extract each feature's cluster assignment from the rsa_cm_res object with extract_cluster(), then run pattern-specified GRSA per cluster with GRSA_by_cm() to obtain enriched GO/pathway terms driven by the cluster-defined expression pattern. The rationale is that GRSA is a threshold-free, multi-group-compatible method, so full (unprefiltered) abundance tables should be supplied; clustering establishes the longitudinal pattern that replaces a two-group contrast. Correct application is judged by biologically coherent clusters, e.g., a cluster expressed only at the differentiated cardiomyocyte stage (day 30) enriching 'Heart process (GO:0003015)'.

## Related tools

- **ReporterScore (R package)** (Provides c_means()/RSA_by_cm(), extract_cluster(), and GRSA_by_cm() for fuzzy c-means clustering and cluster-pattern enrichment) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Dependency installed prior to ReporterScore when using the GitHub development version) — github.com/Asa12138/pcutils
- **devtools** (Used to install ReporterScore and pcutils from GitHub)

## Examples

```
devtools::install_github("Asa12138/pcutils"); devtools::install_github("Asa12138/ReporterScore"); rsa_cm_res <- c_means(ko_abundance, k_num = 6, filter_var = 0.9)
```

## Evaluation signals

- extract_cluster() on rsa_cm_res yields a non-empty, mutually consistent assignment of features to each of the k_num clusters
- The cluster restricted to the differentiated cardiomyocyte stage (day 30, Cluster 6) enriches 'Heart process (GO:0003015)' among its GO terms
- Input abundance table was NOT prefiltered (full background retained) before clustering and GRSA
- Compiled per-cluster enrichment table (cluster_go_enrichment.csv) covers every cluster and lists enriched terms with adjusted p-values
- Metadata rownames match abundance table colnames (all(rownames(metadata) %in% colnames(abundance)) is TRUE)

## Limitations

- Cluster number k_num and filter_var must be chosen manually; no gold standard exists for validating enrichment results, so biological interpretation needs care
- GRSA/clustering enrichment exhibits slightly lower specificity than thresholded methods, requiring additional experimental validation
- Standardization of feature IDs (e.g., gene symbols to K numbers or C numbers) is required before clustering; mismatched sample names between metadata and the abundance table will break the analysis

## Evidence

- [other] Fit fuzzy c-means clusters on the KO abundance table with c_means()/RSA_by_cm(), specifying the number of clusters k_num and filter_var to drop low-variance features.: [evidence span withheld — anchor status: not_found]
- [other] Extract individual cluster assignments from the rsa_cm_res object with extract_cluster().: [evidence span withheld — anchor status: not_found]
- [other] GRSA on the cardiomyocyte differentiation clusters identified 'Heart process (GO:0003015)' as a significantly enriched GO term for Cluster 6: [evidence span withheld — anchor status: not_found]
- [readme] Importantly, the input abundance table should not be prefiltered to retain the background information, as the ‘GRSA’ is a threshold-free method.: [evidence span withheld — anchor status: external_doc]
- [methods] devtools::install_github("Asa12138/ReporterScore"): "devtools::install_github("Asa12138/ReporterScore")"
- [discussion] GRSA considers all KOs involved in the pathway compared to hypergeometric tests that only consider a pre-defined list: "GRSA considers all KOs involved in the pathway compared to hypergeometric tests that only consider a pre-defined list"
