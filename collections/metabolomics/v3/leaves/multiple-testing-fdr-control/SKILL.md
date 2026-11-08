---
name: multiple-testing-fdr-control
description: Use when a pathway-level test yields one p-value per pathway across many
  tested pathways (e.g., NetGSA, DEGraph, CAMERA, Pathway-Express) and you must adjust
  for multiplicity with FDR q-values before calling pathways dysregulated.
when_to_use_negative:
- A single pre-specified pathway hypothesis is being tested, so no multiplicity correction
  is needed
- Family-wise error rate control (e.g., Bonferroni) is required by the study design
  instead of FDR
- The analysis goal is differential gene-level expression, not pathway-level enrichment
  p-values
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: netgsa
  role: R package whose NetGSA function produces the per-pathway p-values that are
    FDR-adjusted; lklMethod='REHE' with sampling is the configured test
  repo: github.com/mikehellstern/netgsa
- name: Pathway-Express
  role: Competing topology-based method whose no-cutoff p-values are passed through
    the same FDR alpha=0.05 detection rule in the power benchmark
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:346c2680d85208ed3602c1fb474c666f1263a0e2bf92ea4fee93fb6fb4944677
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - Pathway-Express
  - Reactome
  - knitr
  - netgsa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# multiple-testing-fdr-control

## Summary

Applies false discovery rate control to the p-values produced by topology-based pathway enrichment testing (e.g., NetGSA with REHE across 114 KEGG pathways), so that pathway detection decisions and empirical power estimates are calibrated at a declared FDR threshold (alpha=0.05).

## When to use

Use when a pathway-level test yields one p-value per pathway across many tested pathways (e.g., NetGSA, DEGraph, CAMERA, Pathway-Express) and you must adjust for multiplicity with FDR q-values before calling pathways dysregulated.

## When NOT to use

- A single pre-specified pathway hypothesis is being tested, so no multiplicity correction is needed
- Family-wise error rate control (e.g., Bonferroni) is required by the study design instead of FDR
- The analysis goal is differential gene-level expression, not pathway-level enrichment p-values

## Inputs

- Vector/table of unadjusted pathway p-values from NetGSA[['results']] (test statistics, p-values per pathway)
- Unadjusted pathway p-values from competing topology-based methods (DEGraph, CAMERA, Pathway-Express, no cutoff)
- Simulation design defining groups by number of dysregulated genes per pathway

## Outputs

- pathway_tests_results.csv — test statistics, p-values, and FDR q-values for all tested pathways (114 KEGG pathways)
- fig5b_power_table.csv — methods x dysregulated-gene-count groups empirical power table with standard errors
- Set of pathways called dysregulated at FDR alpha=0.05

## How to apply

Fit the enrichment model first — in the netgsa workflow, NetGSA is run with lklMethod='REHE' and sampling=TRUE (sample_n=0.25, sample_p=0.25) — then extract pathway_tests[['results']], the table of test statistics, p-values, and FDR q-values for all tested pathways (114 KEGG pathways in the breast cancer application). Apply an FDR cutoff of alpha=0.05: a pathway is classified as detected when its p-value passes the FDR alpha=0.05 threshold. When benchmarking against competing topology-based methods (DEGraph, CAMERA, Pathway-Express), classify detections for each method with the same alpha=0.05 rule and compute empirical power as the fraction of true dysregulated pathways recovered within each number-of-dysregulated-genes group. The rationale is the paper's claim that the overhauled NetGSA preserves statistical power while running in minutes, so FDR-controlled power comparisons are the validation axis, not raw runtime.

## Related tools

- **netgsa** (R package whose NetGSA function produces the per-pathway p-values that are FDR-adjusted; lklMethod='REHE' with sampling is the configured test) — github.com/mikehellstern/netgsa
- **Pathway-Express** (Competing topology-based method whose no-cutoff p-values are passed through the same FDR alpha=0.05 detection rule in the power benchmark)

## Examples

```
pathway_tests <- NetGSA(x, y, beta, A, lklMethod='REHE'); results <- pathway_tests[['results']]; results$qval <- p.adjust(results$pval, method='BH')
```

## Evaluation signals

- The results table must contain p-value and FDR columns for all tested pathways (all 114 KEGG pathways)
- Every pathway detection decision is made against the declared FDR threshold alpha=0.05, uniformly across methods
- fig5b_power_table.csv reproduces the Fig 5B pattern: NetGSA.REHE.Cl competitive with the top method in all groups, DEGraph stronger at low dysregulation, CAMERA/Pathway-Express stronger at high dysregulation
- Empirical power equals the fraction of true dysregulated pathways recovered within each number-of-dysregulated-genes group, with standard errors across simulation replicates
- The full netgsa pipeline (including the FDR q-value output) completes on the order of minutes, consistent with ~2 minutes on a 2-CPU, 16 GB RAM computer

## Limitations

- NetGSA's main function no longer checks variable compatibility between adjacency matrices and input data for block-diagonal structures; this check 'should be added later' per the README
- Before running NetGSA, the input data matrix must be filtered to keep only genes belonging to at least one tested pathway
- There is no consistent best topology-based method for a given problem, so FDR-level power can vary by dysregulation level and method

## Evidence

- [abstract] Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
- [abstract] Existing software tools for topology-based pathway enrichment analysis are either computationally inefficient, have undesirable statistical power, or require expert knowledge: "Existing software tools for topology-based pathway enrichment analysis are either computationally inefficient, have undesirable statistical power, or require expert knowledge"
- [intro] Many topology-based methods have been proposed in the literature, but there is no consistent best choice for a given problem.: "Many topology-based methods have been proposed in the literature, but there is no consistent best choice for a given problem."
- [readme] This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes, as well as novel interactions learned from data.: "This package carries out Network-based Gene Set Analysis by incorporating external information about interactions among genes, as well as novel interactions learned from data."
