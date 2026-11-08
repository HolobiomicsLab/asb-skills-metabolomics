---
name: ora-sensitivity-curve-visualization
description: Use when you want to show how the composition of the differential metabolite
  list (thresholded at BH FDR or Bonferroni q<=0.005/0.05/0.1) affects ORA SEP counts
  across metabolomics datasets.
when_to_use_negative:
- When you only have a single fixed DA metabolite list (e.g. one q <= 0.05 selection)
  and do not intend to sweep threshold or composition variants — ORA should be run
  directly instead of sweeping the list.
- When the goal is to compare pathway databases (KEGG, Reactome, BioCyc) or simulate
  metabolite misidentification; those are separate parameter-effect analyses in the
  same study and are not this sensitivity-curve workflow.
- When the background set itself is under investigation (defining the ideal assay-specific
  background remains an open problem in the source study, so sweeping the DA list
  will not resolve it).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Python 3.8
  role: Runtime for the reproducible_simulations.ipynb notebook that generates and
    plots the sensitivity curves
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test (hypergeometric) used to compute pathway
    p-values for each candidate DA metabolite list before SEP counting
- name: Jupyter Notebook
  role: Hosts reproducible_simulations.ipynb, the code that tabulates SEP counts and
    plots the per-dataset sensitivity curves
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same reproducible_simulations.ipynb
    notebook without local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:6a7966d3fa55d2e331b29d38a06f66e4c43e0971fb33c3ca249c4b37d7c83d37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ora-sensitivity-curve-visualization

## Summary

A sensitivity analysis and visualization skill that plots how the number of significantly enriched pathways (SEPs) changes as the list of differentially abundant metabolites used as ORA input is varied, revealing rise-then-fall behaviour and single-compound-driven extrema.

## When to use

Use when you want to show how the composition of the differential metabolite list (thresholded at BH FDR or Bonferroni q<=0.005/0.05/0.1) affects ORA SEP counts across metabolomics datasets.

## When NOT to use

- When you only have a single fixed DA metabolite list (e.g. one q <= 0.05 selection) and do not intend to sweep threshold or composition variants — ORA should be run directly instead of sweeping the list.
- When the goal is to compare pathway databases (KEGG, Reactome, BioCyc) or simulate metabolite misidentification; those are separate parameter-effect analyses in the same study and are not this sensitivity-curve workflow.
- When the background set itself is under investigation (defining the ideal assay-specific background remains an open problem in the source study, so sweeping the DA list will not resolve it).

## Inputs

- Processed metabolomics matrices (missing values imputed as half of each metabolite's minimum, log2-transformed, auto-scaled)
- Condition-group labels for two-tailed Student's t-tests between groups
- Pathway sets (KEGG, Reactome, BioCyc) restricted to pathways containing at least 3 compounds
- Background/reference compound set for hypergeometric ORA

## Outputs

- sep_counts_table.csv tabulating significant enriched pathway counts by correction method (BH FDR / Bonferroni) and alpha threshold
- Per-dataset sensitivity curves of SEP count versus DA metabolite list construction (cumulative one-at-a-time additions in t-test p-value order or q-thresholded lists)

## How to apply

Load the imputed, log2-transformed and auto-scaled metabolomics matrices and compute two-tailed Student's t-tests between condition groups for every metabolite. Construct candidate differential-metabolite lists in two ways: cumulative lists that add one compound at a time in ascending t-test p-value order, and lists thresholded on BH FDR or Bonferroni-adjusted p-values at q = 0.005, 0.05 and 0.1 (default q <= 0.05). Run right-tailed Fisher's exact (hypergeometric) ORA with scipy.stats.fisher_exact on each candidate list, restricted to pathways containing at least 3 compounds and only computing a pathway p-value when k >= 1 DA metabolite maps to it. Count SEPs at p <= 0.1 without multiple testing correction for each list and dataset, tabulate them in sep_counts_table.csv, and plot per-dataset curves of SEP count against DA list construction in reproducible_simulations.ipynb. The expected output is a rise-then-fall curve that peaks as metabolites are added and falls to zero once the DA list equals the background set, with local extrema attributable to single compounds, and an ordering of Bonferroni < BH FDR in SEP counts whose maximum sits at q <= 0.05.

## Related tools

- **Python 3.8** (Runtime for the reproducible_simulations.ipynb notebook that generates and plots the sensitivity curves)
- **scipy stats fisher_exact** (Right-tailed Fisher's exact test (hypergeometric) used to compute pathway p-values for each candidate DA metabolite list before SEP counting)
- **Jupyter Notebook** (Hosts reproducible_simulations.ipynb, the code that tabulates SEP counts and plots the per-dataset sensitivity curves) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same reproducible_simulations.ipynb notebook without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing

## Examples

```
jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Each sensitivity curve shows a global maximum in SEP count as DA metabolites are added one at a time and falls to zero once the DA list equals the entire background set.
- Local extrema in each curve are attributable to individual compounds (single-compound effects).
- sep_counts_table.csv reproduces the Bonferroni < BH FDR ordering of SEP counts across correction methods.
- The maximum SEP count for BH FDR occurs at the alpha = 0.05 threshold (among 0.005, 0.05, 0.1).
- Only pathways with >= 3 compounds are included in the ORA input, and a pathway p-value is only computed when k >= 1 DA metabolite maps to that pathway.

## Limitations

- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs, so absolute SEP counts are not directly comparable to fully-corrected ORA runs.
- There is no ground-truth dataset where the identities of enriched pathways are known, so curve extrema cannot be validated as biologically correct.
- Curves are produced from five MS-based untargeted metabolomics datasets; generalisation to other data types or platforms is not demonstrated.

## Evidence

- [results] Cumulative DA metabolite lists in t-test p-value order drive SEP count to a global maximum then to zero: [evidence span withheld — anchor status: not_found]
- [discussion] ORA restricted to pathways with at least 3 compounds using right-tailed Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [readme] Sensitivity outputs are tabulated and plotted in the reproducible notebook: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
- [discussion] Limitation: SEP threshold p <= 0.1 without multiple testing correction: [evidence span withheld — anchor status: not_found]
- [abstract] Parameter changes including differential metabolite selection methods alter ORA results: "changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results"
