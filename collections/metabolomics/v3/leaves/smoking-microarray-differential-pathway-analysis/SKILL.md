---
name: smoking-microarray-differential-pathway-analysis
description: Use when you have a smoking/cohort gene expression matrix over predefined
  KEGG or Biocarta pathways and need pathway activity levels and differential ranking
  between current, former, and never smokers.
when_to_use_negative:
- The analysis target is differential expression of individual genes rather than predefined
  pathway activity levels — PLAGE results are complementary to, not a replacement
  for, gene-focused analyses
- The input lacks pre-defined pathway membership sets (KEGG/Biocarta) matching the
  article's ~400 pathway collection
- The dataset is not an expression matrix over individual samples (e.g. already-aggregated
  summary statistics), since SVD requires the pathway-restricted gene-by-sample matrix
  Y
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway activity scoring and permutation
    testing used to produce the smoking study results
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:f3b550e0076370af46b51a1d679a3ca07c59426bb96bb34c45a1ffdaf3bc237b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# smoking-microarray-differential-pathway-analysis

## Summary

Reproduces the PLAGE (Pathway Level Analysis of Gene Expression) analysis of cigarette-smoke airway epithelial expression data (75 subjects: 34 current, 18 former, 23 never smokers), converting ~400 KEGG/Biocarta pathways into SVD-derived activity levels and ranking them by two-group t-statistics with a 10,000-permutation max-t correction.

## When to use

Use when you have a smoking/cohort gene expression matrix over predefined KEGG or Biocarta pathways and need pathway activity levels and differential ranking between current, former, and never smokers.

## When NOT to use

- The analysis target is differential expression of individual genes rather than predefined pathway activity levels — PLAGE results are complementary to, not a replacement for, gene-focused analyses
- The input lacks pre-defined pathway membership sets (KEGG/Biocarta) matching the article's ~400 pathway collection
- The dataset is not an expression matrix over individual samples (e.g. already-aggregated summary statistics), since SVD requires the pathway-restricted gene-by-sample matrix Y

## Inputs

- AGED cigarette smoke airway epithelia gene expression data for 75 subjects (34 current, 18 former, 23 never smokers)
- Pathway definitions for ~400 KEGG and Biocarta pathways (versions used available at the PLAGE website)
- Smoking status group labels for each sample

## Outputs

- table2_top_pathways.tsv with top-ranked pathways and permutation p-values per comparison
- Pathway activity level matrix (first-row SVD coefficients cj of C per pathway per sample)
- Per-comparison ranked pathway significance lists (current vs never, current vs former, former vs never)

## How to apply

Load the AGED airway epithelial smoking dataset (75 subjects) and standardize each gene's expression to zero mean and unit variance over samples. For each of the ~400 predefined KEGG/Biocarta pathways, restrict the expression matrix Y to pathway genes and compute its singular value decomposition Y = WDC, taking the first-row coefficients cj of C as the pathway activity levels for each sample. For each pairwise comparison (current vs never, current vs former, former vs never), compute a two-group t statistic on mean pathway activity. Assess significance with 10,000 random permutations of sample labels, recording the maximum t-statistic across pathways per permutation, and compute each pathway's p-value as the fraction of maximum t-statistics exceeding the observed one. Rank pathways by significance within each comparison and write the top-ranked pathways with p-values to table2_top_pathways.tsv; validation requires 'gamma-Hexachlorocyclohexane degradation' (KEGG) as the top pathway in both current-vs-never and former-vs-never comparisons.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway activity scoring and permutation testing used to produce the smoking study results) — http://dulci.biostat.edu/pathways

## Examples

```
import numpy as np; Y = standardized_pathway_matrix; W, D, C = np.linalg.svd(Y); activity = C[0]  # first metagene levels = PLAGE pathway activity
```

## Evaluation signals

- Top-ranked pathway in both current-vs-never and former-vs-never comparisons must be 'gamma-Hexachlorocyclohexane degradation' (KEGG), matching Table 2
- p-values must be computed against the null of 10,000 max-t statistics, not naive per-pathway permutation p-values
- Gene expression must be standardized to zero mean and unit variance per gene over samples before SVD
- Exactly 75 samples (34 current, 18 former, 23 never) should enter the analysis
- Pathway count should approximate the article's ~400 KEGG/Biocarta pathways and complexes

## Limitations

- The specific KEGG and Biocarta pathway versions used are only available via the PLAGE website; no version numbers or accessions are stated, so pathway set drift can affect reproduction
- The t-statistic formula is referenced via an equation placeholder in the article, so the exact statistic definition must be inferred
- The PLAGE web program repository URL (http://dulci.biostat.edu/pathways) may no longer be live, requiring reimplementation

## Evidence

- [intro] SVD pathway activity derivation: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] Permutation scheme: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] Pathway universe: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [abstract] Smoking application findings: "A number of interesting pathways are identified in comparisons between smokers and non-smokers including ones related to nicotine metabolism, mucus production, and glutathione metabolism."
