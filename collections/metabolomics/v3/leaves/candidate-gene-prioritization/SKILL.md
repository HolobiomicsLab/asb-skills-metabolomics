---
name: candidate-gene-prioritization
description: Use when a Bayesian direction-of-effect model identifies traits with
  confident positive enrichments or depletions and you need to name the concrete genes
  (e.g. Fig. 6 candidates) driving those enrichments across time points and tissues.
when_to_use_negative:
- When the goal is genome-wide heritability partitioning (LDSC/MESC hSNP2 estimation)
  rather than naming specific exercise-by-disease candidate genes
- When no S-PrediXcan output or PrediXcan-DEG intersecting gene sets exist for the
  trait of interest — direction signs and effect sizes cannot be assigned
- When qualitative sex-specificity of exercise adaptation is the target; the article
  explicitly states this 'fell outside the scope considered here'
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3170
tools:
- name: S-PrediXcan
  role: Supplies gene-by-tissue-level associations and directions of effect for specific
    diseases used to sign candidate genes
  repo: https://github.com/hakyimlab/MetaXcan
- name: CmdStanR / Stan
  role: Fits the binomial Bayesian multilevel model whose posterior means of direction-of-effect
    deviations drive trait ranking and candidate selection
- name: DESeq2
  role: Per-tissue, per-sex differential expression analysis whose log2FC signs enter
    the candidate gene direction sign
- name: Matrix (nearPD)
  role: Projects the G_SNP correlation matrix (from GCTA-GRM estimates) to positive
    semi-definiteness for the Stan model underlying the ranking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:a6f8d634772b64273d7e18e70339ec6243d61b9ec06fbd4b4b76269f5ab0e1ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hakyimlab/MetaXcan
  tools:
  - CmdStanR
  - CmdStanR / Stan
  - DESeq2
  - GCTA-GRM
  - Matrix (nearPD)
  - Plink
  - S-PrediXcan
  - Stan
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# candidate-gene-prioritization

## Summary

Prioritizes trait-specific candidate genes at the intersection of exercise-induced differential expression and PrediXcan disease associations, by tracing direction-of-effect posterior means across training time points and compiling genes with tissue, +/- direction sign (product of DE and PrediXcan association directions), and standardized effect size.

## When to use

Use when a Bayesian direction-of-effect model identifies traits with confident positive enrichments or depletions and you need to name the concrete genes (e.g. Fig. 6 candidates) driving those enrichments across time points and tissues.

## When NOT to use

- When the goal is genome-wide heritability partitioning (LDSC/MESC hSNP2 estimation) rather than naming specific exercise-by-disease candidate genes
- When no S-PrediXcan output or PrediXcan-DEG intersecting gene sets exist for the trait of interest — direction signs and effect sizes cannot be assigned
- When qualitative sex-specificity of exercise adaptation is the target; the article explicitly states this 'fell outside the scope considered here'

## Inputs

- Posterior MCMC output from the CmdStanR binomial Bayesian multilevel model (/data/MCMC_output) with trait x tissue direction-of-effect deviations
- Standardized exercise effect sizes for the intersecting PrediXcan-DEG gene sets
- 8-week sex-consistent gene set (F1_M1 and F-1_M-1 repfdr states) and 1-, 2-, and 4-week differential expression assignments
- S-PrediXcan gene-by-tissue association directions and effect sizes (IHW alpha = 0.05, tissue x trait pairs as covariate)

## Outputs

- Ranked table of traits by marginal posterior mean direction-of-effect deviation (strongest positive: body fat percentage, asthma, BMI; strongest depletion: standing height, high cholesterol)
- Figure 6 candidate gene table with columns: gene, trait (high cholesterol / asthma), tissue, +/- direction sign (DE direction x PrediXcan association direction), standardized effect size
- Time-course trace of proportion of positive effects from 8 weeks back through 4, 2, and 1 weeks

## How to apply

Load the posterior MCMC output (/data/MCMC_output) and standardized exercise effect sizes from the MoTrPAC_Complex_Traits repository, then extract marginal posterior means of the direction-of-effect deviation per trait x tissue and rank traits; the expected pattern is strongest positive enrichment for body fat percentage, asthma, and BMI and strongest depletion for standing height and high cholesterol. For the two non-anthropometric traits with the highest posterior means (high cholesterol and asthma), trace the proportion of positive effects of the 8-week sex-consistent gene set (F1_M1 / F-1_M-1 states) backwards through the 1-, 2-, and 4-week time points, assigning 0.5 to genes not assigned to a differential-expression node. Compile the candidate gene table listing cholesterol genes (NDUFA13, FADS2, PNKD, AAMP, OGDH, TMBIM1, APOB, ABCG8) and asthma genes (BAG6, CCNG, CRAT, PTPA, FAM89B, ATP6V1G2, ENDOU, CCNF), recording for each row the tissue, direction sign computed as the product of DE direction and PrediXcan association direction, and standardized effect size consistent with Fig. 2b. Validity rests on the underlying filters: training-differential genes at 5% IHW alpha and S-PrediXcan hits at IHW alpha = 0.05 with tissue x trait pairs as covariate.

## Related tools

- **S-PrediXcan** (Supplies gene-by-tissue-level associations and directions of effect for specific diseases used to sign candidate genes) — https://github.com/hakyimlab/MetaXcan
- **CmdStanR / Stan** (Fits the binomial Bayesian multilevel model whose posterior means of direction-of-effect deviations drive trait ranking and candidate selection)
- **DESeq2** (Per-tissue, per-sex differential expression analysis whose log2FC signs enter the candidate gene direction sign)
- **Matrix (nearPD)** (Projects the G_SNP correlation matrix (from GCTA-GRM estimates) to positive semi-definiteness for the Stan model underlying the ranking)

## Examples

```
Rscript direction_of_effect.R --mcmc /data/MCMC_output --effects standardized_effect_sizes.rds --chains 4 --warmup 2500 --sampling 2500 --adapt_delta 0.95
```

## Evaluation signals

- Direction-of-effect posterior means reproduce the reported trait ranking: strongest positive enrichments for body fat percentage, asthma, and BMI; strongest depletions for standing height and high cholesterol
- Every candidate gene row carries a tissue, a +/- direction sign (product of DE direction and PrediXcan association direction), and a standardized effect size consistent with Fig. 2b
- Cholesterol candidate list is exactly NDUFA13, FADS2, PNKD, AAMP, OGDH, TMBIM1, APOB, ABCG8; asthma list is BAG6, CCNG, CRAT, PTPA, FAM89B, ATP6V1G2, ENDOU, CCNF
- Genes not assigned to a differential-expression node receive a 0.5 proportion-positive value in the time-course trace
- Convergence diagnostics of the upstream Stan model pass: rhat < 1.01, bulk/tail ESS > 500, < 0.05% divergences

## Limitations

- Effect sizes are small: enrichment/depletion varies predominately within 0 ± 0.3 on the log-odds scale (≈7.5% maximum difference on the probability scale)
- Some marginal posterior distributions for difference parameters did not reach the 95% posterior mass threshold; no individual tissues were confidently enriched in positive effects for asthma
- Rat-to-human transfer (≈140 million years of evolution) and GTEx v8 age mismatch (humans 50+ vs rats under eight months) limit interpretability of prioritized candidates
- Results may have limited portability to non-European populations, as the GTEx sample comprises mostly European-descendant individuals

## Evidence

- [other] the strongest confident mean enrichments for positive effects were observed in body fat percentage, asthma, and body mass index (BMI), and the strongest mean depletions in standing height and high cholesterol: "the strongest confident mean enrichments for positive effects were observed in body fat percentage, asthma, and body mass index (BMI), and the strongest mean depletions in standing height and high"
- [intro] S-PrediXcan direction-of-effect data: "we leveraged published S-PrediXcan12,13 output, which estimates gene by tissue-level associations and directions of effect for specific diseases"
- [methods] Selection of sex-consistent 8-week state genes: [evidence span withheld — anchor status: not_found]
- [methods] Filtering S-PrediXcan hits: "filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate)"
- [discussion] Small effect sizes caveat: [evidence span withheld — anchor status: not_found]
