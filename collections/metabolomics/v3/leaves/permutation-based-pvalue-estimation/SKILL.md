---
name: permutation-based-pvalue-estimation
description: Use when testing many predefined gene-sets (~400 KEGG/Biocarta pathways)
  for differential activity between two sample groups and you need a multiple-testing-aware
  empirical p-value without parametric assumptions on small microarray cohorts.
when_to_use_negative:
- When the number of possible distinct sample-label permutations is too small to support
  the permutation count (e.g. very few samples per group), since the empirical p-value
  resolution is bounded by the permutation space
- When testing a single pre-specified hypothesis rather than the full ~400-pathway
  collection — the max-statistic null would needlessly apply a collection-wide multiple-testing
  correction
- When a validated parametric correction (e.g. FDR over computed statistics) is explicitly
  required by the analysis plan
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program implementing the pathway-level analysis whose significance is
    assessed by 10,000 sample-label permutations recording the most significant pathway's
    t-statistic
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:51fadb6a91d0681622b15761b701a61dba66f45af763ea8017e3d6e288489c45
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# permutation-based-pvalue-estimation

## Summary

Estimates pathway-level significance by randomly permuting sample labels many times (10,000 in the PLAGE article; 1,000 for the GSEA-style ES workflow) and building a null distribution from the most extreme statistic (max t-statistic or max enrichment score) across all ~400 KEGG/Biocarta gene-sets, so that p-values account for testing hundreds of pathways simultaneously.

## When to use

Use when testing many predefined gene-sets (~400 KEGG/Biocarta pathways) for differential activity between two sample groups and you need a multiple-testing-aware empirical p-value without parametric assumptions on small microarray cohorts.

## When NOT to use

- When the number of possible distinct sample-label permutations is too small to support the permutation count (e.g. very few samples per group), since the empirical p-value resolution is bounded by the permutation space
- When testing a single pre-specified hypothesis rather than the full ~400-pathway collection — the max-statistic null would needlessly apply a collection-wide multiple-testing correction
- When a validated parametric correction (e.g. FDR over computed statistics) is explicitly required by the analysis plan

## Inputs

- Gene expression matrix for a two-group comparison (e.g. type 2 diabetes expression data or airway epithelia data from smokers and non-smokers)
- Collection of predefined pathways/complexes (~400 KEGG and Biocarta gene-sets)
- Per-pathway observed statistics (t-statistic from PLAGE pathway activity levels, or GSEA enrichment score from signal-to-noise gene rankings)

## Outputs

- Per-pathway empirical permutation p-values
- Null distribution of maximum statistics across permutations
- Table of all pathway statistics and p-values (e.g. gsea_results.csv)

## How to apply

After computing a statistic for each pathway (a PLAGE t-statistic on pathway activity levels, or a GSEA enrichment score from signal-to-noise gene rankings), generate a null by randomly permuting sample labels a large number of times — 10,000 permutations in the PLAGE article, 1,000 in the GSEA-style variant. For each permutation, recompute the statistic for every pathway and record only the most extreme value across the full collection (the max t-statistic or max ES). This maximum-statistic scheme couples the pathways so the resulting p-value — the fraction of permutation maxima exceeding the observed statistic — is corrected for the fact that ~400 gene-sets and complexes were tested jointly. Write the observed statistics and the permutation p-values for all pathways to an output table (e.g. gsea_results.csv) for downstream reporting.

## Related tools

- **PLAGE** (Web program implementing the pathway-level analysis whose significance is assessed by 10,000 sample-label permutations recording the most significant pathway's t-statistic) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Every p-value equals the fraction of permutation maxima exceeding the observed statistic and is bounded below by 1/number_of_permutations (e.g. ≥ 1/10000 for PLAGE, ≥ 1/1000 for the GSEA-style workflow)
- Replication of the smoke-data comparison: current vs. never smokers places KEGG ribosome at the highest ES (~162) with p ≈ 0.17
- Replication of the smoke-data comparison: former vs. never smokers gives ribosome ES ~304 with p ≈ 0.047
- Increasing the permutation count from 1,000 toward 10,000 should stabilize p-values rather than systematically shift them
- The null distribution should be built from the maximum statistic across all ~400 gene-sets per permutation, not per-pathway nulls, confirming the multiple-testing coupling

## Limitations

- The number of permutations trades runtime against p-value resolution; the article used 10,000, and the GSEA-style workflow 1,000, so p-values near the minimum (e.g. 0.001) are poorly resolved at 1,000 permutations
- KEGG and Biocarta pathways continually expand and evolve; the versions used for the published results are only available at the PLAGE website and no version numbers or accessions are stated
- The max-statistic approach yields conservative p-values since the null is driven by the single most significant pathway in each permutation

## Evidence

- [discussion] Permutation null across the pathway collection: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] Pairwise comparison framing: "we mainly perform pairwise comparisons to identify pathways for which the mean activity level in one group (e.g., diabetic) is significantly different from that in the other"
- [discussion] Pathway collection tested: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [abstract] Implementation vehicle: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
