---
name: exercise-deg-time-course-trajectory-tracing
description: Use when a trait x tissue gene set needs its direction-of-effect pattern
  decomposed across the 1-, 2-, 4-, and 8-week MoTrPAC endurance exercise training
  time points to see when the disease-like shift emerged.
when_to_use_negative:
- The dataset has only a single time point — trajectory tracing requires the paired
  1-, 2-, 4-, and 8-week MoTrPAC time course to be meaningful
- The goal is estimating SNP-heritability or expression-mediated heritability (hSNP2)
  — use LDSC/MESC instead, which operate on GWAS/eQTL summary statistics rather than
  DE trajectories
- Genes lack expression values across some conditions — the z-score matrix was explicitly
  filtered to genes with no missing values across all conditions, so those genes must
  be excluded rather than imputed with signs
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Likelihood-ratio-test differential expression per sex and tissue, producing
    the p-value/log2FC pairs that become time-course z-scores
- name: repfdr
  role: EM clustering of cross-condition z-scores into simplified states (F1_M1, F-1_M-1)
    that define the differential-expression nodes traced across weeks
- name: MoTrPAC_Complex_Traits scripts
  role: Repository scripts that implement the trajectory tracing and candidate-gene
    compilation from posterior MCMC and standardized effect sizes
  repo: https://github.com/NikVetr/MoTrPAC_Complex_Traits
- name: S-PrediXcan
  role: Provides gene-by-tissue association directions of effect whose sign is multiplied
    with the DE direction to define positive vs negative effects
  repo: https://github.com/hakyimlab/MetaXcan
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:e860bd1ad9c583e3afda5cab912b974945d9912089a9c96069c7b15568a8a04d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NikVetr/MoTrPAC_Complex_Traits
  tools:
  - CmdStanR
  - DESeq2
  - GCTA-GRM
  - MoTrPAC_Complex_Traits scripts
  - Plink
  - S-PrediXcan
  - Stan
  - repfdr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# exercise-deg-time-course-trajectory-tracing

## Summary

Traces the proportion of positive exercise-induced differential-expression effects backwards from the 8-week sex-consistent gene set through the 1-, 2-, and 4-week MoTrPAC rat training time points, using repfdr-assigned DE states and imputing 0.5 for genes not assigned to a differential-expression node.

## When to use

Use when a trait x tissue gene set needs its direction-of-effect pattern decomposed across the 1-, 2-, 4-, and 8-week MoTrPAC endurance exercise training time points to see when the disease-like shift emerged.

## When NOT to use

- The dataset has only a single time point — trajectory tracing requires the paired 1-, 2-, 4-, and 8-week MoTrPAC time course to be meaningful
- The goal is estimating SNP-heritability or expression-mediated heritability (hSNP2) — use LDSC/MESC instead, which operate on GWAS/eQTL summary statistics rather than DE trajectories
- Genes lack expression values across some conditions — the z-score matrix was explicitly filtered to genes with no missing values across all conditions, so those genes must be excluded rather than imputed with signs

## Inputs

- MoTrPAC rat EET differential-expression results per sex and tissue at 1, 2, 4, and 8 training weeks (DESeq2 LRT p-values and log2FC)
- repfdr EM state assignments (simplified F1_M1 / F-1_M-1 states) per gene per time point
- Standardized exercise effect sizes for the PrediXcan–DEG intersecting gene sets
- S-PrediXcan gene-by-tissue association directions used to form the sign product with DE direction

## Outputs

- Per-gene time-course trajectory of differential-expression state and sign across 1-, 2-, 4-, and 8-week nodes (0.5 imputed for non-DE genes)
- Proportion-positive summary per trait x tissue at each training duration
- Candidate gene table with tissue, +/- direction sign, and standardized effect size (e.g. NDUFA13, FADS2 for cholesterol; BAG6, CCNG for asthma)

## How to apply

Start from the 8W sex-consistent gene set — genes assigned by the repfdr EM procedure to the F1_M1 (up-regulated in both females) or F-1_M-1 (down-regulated in both females) states — and compute the proportion of effects whose sign is positive (product of DE direction and PrediXcan association direction). Backtrack through the 1-, 2-, and 4-week time points of the same MoTrPAC EET differential-expression calls, carrying each gene's sign through time; for genes not assigned to a differential-expression node at a time point, assign 0.5 (uninformative) rather than dropping them, so proportions remain comparable across weeks. Inputs to the state assignment are z-scores built as qnorm(p/2, lower.tail = F) * sign(log2FC) from DESeq2 likelihood-ratio-test results (male and female p-values combined via Fisher's sum of logs), filtered to genes with no missing values across all conditions. The output trajectory table feeds the candidate gene listings (e.g. cholesterol genes NDUFA13, FADS2, PNKD, AAMP, OGDH, TMBIM1, APOB, ABCG8; asthma genes BAG6, CCNG, CRAT, PTPA, FAM89B, ATP6V1G2, ENDOU, CCNF) with tissue, +/- direction sign, and standardized effect size consistent with Fig. 2b.

## Related tools

- **DESeq2** (Likelihood-ratio-test differential expression per sex and tissue, producing the p-value/log2FC pairs that become time-course z-scores)
- **repfdr** (EM clustering of cross-condition z-scores into simplified states (F1_M1, F-1_M-1) that define the differential-expression nodes traced across weeks)
- **MoTrPAC_Complex_Traits scripts** (Repository scripts that implement the trajectory tracing and candidate-gene compilation from posterior MCMC and standardized effect sizes) — https://github.com/NikVetr/MoTrPAC_Complex_Traits
- **S-PrediXcan** (Provides gene-by-tissue association directions of effect whose sign is multiplied with the DE direction to define positive vs negative effects) — https://github.com/hakyimlab/MetaXcan

## Examples

```
z <- qnorm(p/2, lower.tail = FALSE) * sign(log2FC); state <- repfdr_assign(z); pos <- ifelse(is.na(state), 0.5, sign)
```

## Evaluation signals

- Every traced gene row carries a tissue, a +/- direction sign (product of DE direction and PrediXcan association direction), and a standardized effect size consistent with Fig. 2b
- Genes not assigned to a differential-expression node at a given week contribute 0.5, not an excluded observation — the denominator stays constant across time points
- The 8W starting set contains only genes in the F1_M1 (both-female up) or F-1_M-1 (both-female down) states
- The z-score matrix was filtered to genes with no missing values across all conditions before state assignment
- The traced backward trajectory should explain why the highest posterior-mean non-anthropometric traits (e.g. cholesterol and asthma) show the candidate genes listed in the article

## Limitations

- Rats are separated from humans by nearly 140 million years of evolution, complicating interpretation of conserved direction-of-effect trajectories
- Results may have limited portability to non-European populations, as the GTEx sample used for S-PrediXcan directions comprises mostly European-descendant individuals
- Important biology in disease/exercise-responsive genes without easily established cross-species correspondence is likely missed by this rat-anchored tracing
- No individual tissues were found to be confidently enriched in positive effects for asthma, so trajectory-based candidates for that trait rest on marginal rather than confident enrichments

## Evidence

- [methods] repfdr states: "the expectation-maximization (EM) process of the repfdr algorithm was used to assign one of three simplified states to each z-score"
- [methods] sex-consistent selection: [evidence span withheld — anchor status: not_found]
- [methods] z-score construction: [evidence span withheld — anchor status: not_found]
- [methods] missing-value filter: "The z-score matrix was filtered to include the set of genes that had no missing values across all conditions."
- [other] 0.5 imputation rule: [evidence span withheld — anchor status: not_found]
- [discussion] asthma caveat: "Though no individual tissues were found to be confidently enriched in positive effects for this phenotype"
