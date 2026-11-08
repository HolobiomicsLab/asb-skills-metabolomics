---
name: harmonic-mean-pvalue-aggregation
description: Use when a gene-set or enrichment analysis yields multiple correlated
  subtest p-values (e.g. Bonferroni-adjusted fgsea() results per trait×tissue pair)
  that must be combined into one FWER-controlled test statistic.
when_to_use_negative:
- The subtests being aggregated are statistically independent — then Fisher's sum-of-logs
  method (as the paper used to combine male- and female-specific DE p-values) is the
  more appropriate, more powerful choice.
- You need enrichment estimates per tissue or trait category with directionality and
  partial pooling across correlated tests — the hierarchical Bayesian Stan model (95%
  posterior-mass flagging) is the study's primary tool for that, not p-value aggregation.
- The input is a single test statistic per hypothesis with no interdependent substructure
  — no aggregation is needed.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: fgsea
  role: Runs the frequentist gene set enrichment analysis per trait×tissue pair whose
    subtest p-values are subsequently aggregated by the harmonic mean
- name: S-PrediXcan
  role: Supplies the gene-by-tissue association p-values (IHW α = 0.05 with tissue×trait
    pairs as covariate) that are transformed into the -log10 ranking metric for fgsea()
  repo: github.com/hakyimlab/MetaXcan
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:a62d6f8cabfb4c45d2744da1360faed64eaf69cff2942103000c5d874de516a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CmdStanR
  - DESeq2
  - PrediXcan
  - S-PrediXcan
  - Stan
  - fgsea
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# harmonic-mean-pvalue-aggregation

## Summary

Aggregates p-values from interdependent subtests (e.g. per trait-by-tissue GSEA runs on PrediXcan hits) into a single combined p-value via the harmonic mean, controlling family-wise error rate at α = 0.05 without requiring independence among the aggregated tests.

## When to use

Use when a gene-set or enrichment analysis yields multiple correlated subtest p-values (e.g. Bonferroni-adjusted fgsea() results per trait×tissue pair) that must be combined into one FWER-controlled test statistic.

## When NOT to use

- The subtests being aggregated are statistically independent — then Fisher's sum-of-logs method (as the paper used to combine male- and female-specific DE p-values) is the more appropriate, more powerful choice.
- You need enrichment estimates per tissue or trait category with directionality and partial pooling across correlated tests — the hierarchical Bayesian Stan model (95% posterior-mass flagging) is the study's primary tool for that, not p-value aggregation.
- The input is a single test statistic per hypothesis with no interdependent substructure — no aggregation is needed.

## Inputs

- Per-subtest p-values (Bonferroni-adjusted) from fgsea() GSEA runs on trait×tissue pairs
- -log10(S-PrediXcan p-values) ranking metrics of gene-level associations (IHW α = 0.05 filtered)
- Gene set membership vector (e.g. sex-consistent 8-week exercise DEGs from repfdr states F1_M1 / F-1_M-1)

## Outputs

- Harmonic-mean combined p-value per aggregated hypothesis (e.g. high-cholesterol: adjusted p ≈ 0.064, ES ≈ 0.63)
- Binary enrichment call per hypothesis at FWER α = 0.05

## How to apply

First run frequentist GSEA with fgsea() per trait×tissue pair, using -log10(S-PrediXcan p-values) as the ranking metric of the gene-level association statistics, and apply Bonferroni correction across the subtests. Then, where the same biological hypothesis decomposes into interdependent subtests (e.g. a trait tested in several tissues), combine their adjusted p-values with the harmonic-mean p-value method, which is valid for dependent tests. Declare significance at family-wise error rate α = 0.05; in the source study this surfaced the high-cholesterol test (enrichment score ES ≈ 0.63, harmonic-mean adjusted p ≈ 0.064) as a borderline result reported alongside the confident findings. The rationale is that gene-set tests on the same PrediXcan ranking metric across tissues or trait categories are strongly correlated, so naive aggregation would inflate false positives while the harmonic mean tolerates the dependence structure.

## Related tools

- **fgsea** (Runs the frequentist gene set enrichment analysis per trait×tissue pair whose subtest p-values are subsequently aggregated by the harmonic mean)
- **S-PrediXcan** (Supplies the gene-by-tissue association p-values (IHW α = 0.05 with tissue×trait pairs as covariate) that are transformed into the -log10 ranking metric for fgsea()) — github.com/hakyimlab/MetaXcan

## Examples

```
fgRes <- fgsea(pathways = list(deg8w = sex_consistent_8w_genes), stats = -log10(spx$p.value)); hmp <- min(1, length(fgRes$padj) / sum(1 / fgRes$padj))
```

## Evaluation signals

- The aggregated harmonic-mean p-value must lie between the minimum and maximum of the input subtest p-values (harmonic mean cannot exceed the smallest p-value times a factor bounded by the number of tests).
- Family-wise error rate is controlled at α = 0.05: the number of declared-enriched hypotheses should not exceed what Bonferroni-corrected subtests support (no result declared significant with subtest adjusted p-values far above 0.05).
- Benchmark against the source study: the high-cholesterol test should reproduce ES ≈ 0.63 and harmonic-mean adjusted p ≈ 0.064.
- Input p-values must come from the same ranking metric definition (-log10 PrediXcan p-values) and the same gene universe per trait×tissue cell.
- All input subtest p-values should be valid (in (0,1]) and traceable to Bonferroni correction over the fgsea() runs; zero or NA p-values must be handled before aggregation.

## Limitations

- Harmonic-mean aggregation is conservative when only a minority of subtests are truly significant; borderline results like the high-cholesterol test (adjusted p ≈ 0.064) fail the α = 0.05 FWER threshold even with ES ≈ 0.63.
- Aggregation across trait×tissue cells ignores the correlation structure modeled elsewhere in the study (Σi, Σj, Σi×j matrices estimated under a bivariate probit), so it cannot estimate direction or magnitude of enrichment.
- The harmonic-mean combined p-value does not by itself indicate direction of effect (enrichment vs. depletion); that requires the ranking metric's sign or the Bayesian difference effects.

## Evidence

- [other] Run frequentist Gene Set Enrichment Analysis with fgsea() per trait×tissue pair using -log10(PrediXcan p-values) as the ranking metric with Bonferroni correction: [evidence span withheld — anchor status: not_found]
- [other] aggregate interdependent subtests by harmonic-mean p-values with FWER α = 0.05, including the high-cholesterol test (ES ≈ 0.63, adjusted p ≈ 0.064): [evidence span withheld — anchor status: not_found]
- [methods] filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate): "filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate)"
- [readme] S-PrediXcan is an extension that infers PrediXcan's results using only summary statistics: [evidence span withheld — anchor status: not_found]
