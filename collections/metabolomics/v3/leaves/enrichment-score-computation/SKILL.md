---
name: enrichment-score-computation
description: Use when reproducing or benchmarking GSEA results on the Mootha type
  2 diabetes (17 DM2 / 17 NGT / 9 IGT) or airway epithelia smoker datasets, or when
  validating a pathway-analysis method against published ES values and permutation
  p-values.
when_to_use_negative:
- You want the paper's own SVD-based pathway activity levels compared by mean t-statistic
  — that is the PLAGE method, not the GSEA ES computation being reproduced for comparison
- You lack predefined pathway gene-sets (KEGG/Biocarta); the ES is defined only relative
  to a fixed gene-set collection
- You need per-sample pathway activity scores rather than a per-comparison gene-set
  statistic
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: The SVD-based pathway method whose published results this GSEA ES computation
    is used to compare against; its website hosts the KEGG/Biocarta pathway versions
    used in the article
  repo: http://dulci.biostat.edu/pathways
- name: KEGG
  role: Source of predefined pathway gene-sets (e.g., ribosome, oxidative phosphorylation)
    scored by the ES
- name: Biocarta
  role: Source of predefined pathway gene-sets (e.g., Lectin Induced Complement Pathway)
    scored by the ES
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:0721915c5446585d12a49567256f2bbc68960c5ad0079402317693e7672dc37f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biocarta
  - KEGG
  - PLAGE
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# enrichment-score-computation

## Summary

Reproduces the gene-set enrichment analysis (GSEA) scoring scheme referenced by the PLAGE paper: rank genes by signal-to-noise ratio, compute an enrichment score (ES) for each KEGG/Biocarta pathway from its members' ranks, and estimate p-values by permuting sample labels and comparing against the maximum permuted ES.

## When to use

Use when reproducing or benchmarking GSEA results on the Mootha type 2 diabetes (17 DM2 / 17 NGT / 9 IGT) or airway epithelia smoker datasets, or when validating a pathway-analysis method against published ES values and permutation p-values.

## When NOT to use

- You want the paper's own SVD-based pathway activity levels compared by mean t-statistic — that is the PLAGE method, not the GSEA ES computation being reproduced for comparison
- You lack predefined pathway gene-sets (KEGG/Biocarta); the ES is defined only relative to a fixed gene-set collection
- You need per-sample pathway activity scores rather than a per-comparison gene-set statistic

## Inputs

- Gene expression matrix from the type 2 diabetes study (43 samples: 17 DM2, 17 NGT, 9 IGT) from the Whitehead Institute deposit
- Gene expression matrix from the airway epithelia study (75 samples: 34 current, 18 former, 23 never smokers) from the AGED database
- Predefined pathway gene-set collection (~400 pathways and complexes from KEGG and Biocarta)
- Two-group sample-label assignment for each pairwise comparison

## Outputs

- gsea_results.csv containing enrichment scores and permutation p-values for every gene-set in each comparison
- Ranked gene lists (signal-to-noise ratio) per comparison
- Null distribution of maximum ES values over 1000 permuted label assignments

## How to apply

For each pairwise comparison (current vs never, former vs never, current vs former smokers, DM2 vs NGT), rank all genes by the signal-to-noise ratio — the absolute difference of group means divided by the sum of within-group standard deviations, following the Mootha et al. procedure. Compute the enrichment score for each of the ~400 KEGG/Biocarta pathway gene-sets from its member genes' positions in the ranked list. Permute sample labels 1000 times; at each permutation recompute the rankings and record the maximum ES across all gene-sets to build the null distribution. Each pathway's p-value is the fraction of permuted maximum ES values exceeding the observed ES, and all ES and p-values are written to gsea_results.csv. The permutation-over-maximum-ES scheme mirrors the PLAGE paper's design of permuting sample labels and recording the statistic of the most significant pathway, which is why these outputs serve as its GSEA comparison baseline.

## Related tools

- **PLAGE** (The SVD-based pathway method whose published results this GSEA ES computation is used to compare against; its website hosts the KEGG/Biocarta pathway versions used in the article) — http://dulci.biostat.edu/pathways
- **KEGG** (Source of predefined pathway gene-sets (e.g., ribosome, oxidative phosphorylation) scored by the ES)
- **Biocarta** (Source of predefined pathway gene-sets (e.g., Lectin Induced Complement Pathway) scored by the ES)

## Evaluation signals

- Current vs never smokers: KEGG ribosome gene-set should be the highest ES (~162) with permutation p ≈ 0.17 over 1000 permutations
- Former vs never smokers: ribosome ES ~304 with p ≈ 0.047 — the only comparison where the ribosome set reaches significance at 1000 permutations
- DM2 vs NGT: ribosome ES 278 (p = 0.073) must be the top pathway, with oxidative phosphorylation next at ES 261 (p = 0.092)
- Current vs former smokers: Biocarta Lectin Induced Complement Pathway should yield ES 128 with p = 0.52 (non-significant)
- Reproduced ES and p-values must match reported values within permutation-sampling variability; ribosome should rank highest in three of the four comparisons

## Limitations

- P-values are estimated from only 1000 permutations, so they are subject to permutation-sampling variability (minimum attainable p ≈ 0.001; p = 0.073 and 0.092 are borderline)
- KEGG and Biocarta pathways 'continue to expand and evolve', so pathway-set version drift can change ES values; only the versions at the PLAGE website match the article
- The ES is a per-comparison statistic tied to the signal-to-noise ranking and maximum-ES null, so it is not directly comparable to PLAGE's t-statistic-permutation p-values (10,000 permutations)

## Evidence

- [abstract] A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided.: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [other] rank all genes using the signal-to-noise ratio (absolute difference of group means divided by the sum of within-group standard deviations) following the Mootha et al. procedure: [evidence span withheld — anchor status: not_found]
