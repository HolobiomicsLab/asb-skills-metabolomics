---
name: significant-pathway-count-tabulation
description: Use when running right-tailed Fisher's exact ORA (scipy.stats.fisher_exact)
  on multiple candidate metabolite-of-interest lists or correction/alpha settings
  and you need to tabulate SEP counts to show how those choices affect enrichment
  outcomes.
when_to_use_negative:
- When a single DA-metabolite list and single correction/alpha setting is analysed
  and no sensitivity comparison is needed — the tabulation is only meaningful across
  multiple candidate parameterisations.
- When ORA output is already a final list of significantly enriched pathways with
  BH FDR-adjusted q-values and you only need to report that list, not compare counts
  across settings.
- When pathway p-values have not been computed (e.g. input is a raw feature-abundance
  matrix that has not yet gone through imputation, log2 transformation, auto-scaling,
  t-testing, and Fisher's exact ORA).
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: scipy stats fisher_exact
  role: Computes right-tailed Fisher's exact (hypergeometric) pathway p-values used
    as input to the SEP count tabulation
- name: Python 3.8
  role: Execution environment for the simulations and SEP-count tabulation
- name: Jupyter Notebook
  role: Hosts reproducible_simulations.ipynb, the notebook in which SEP counts are
    tabulated into sep_counts_table.csv and plotted
- name: Google Colaboratory
  role: Alternative remote execution of the same reproducible_simulations.ipynb notebook
    without local installation
  repo: https://research.google.com/colaboratory/faq.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:2039b59a958f41924ed56479e4210ee9229f869ac67689de49156d6e9846d87f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# significant-pathway-count-tabulation

## Summary

Tabulate counts of significantly enriched pathways (SEPs) across candidate DA-metabolite lists, correction methods, and alpha thresholds for ORA sensitivity analysis. This skill quantifies how parameter choices — the metabolite-of-interest list, BH FDR vs Bonferroni adjustment, and the p <= 0.1 SEP cutoff — drive the number of detected pathways, and is implemented in the reproducible_simulations.ipynb notebook of the metabolomics-ORA repository.

## When to use

Use when running right-tailed Fisher's exact ORA (scipy.stats.fisher_exact) on multiple candidate metabolite-of-interest lists or correction/alpha settings and you need to tabulate SEP counts to show how those choices affect enrichment outcomes.

## When NOT to use

- When a single DA-metabolite list and single correction/alpha setting is analysed and no sensitivity comparison is needed — the tabulation is only meaningful across multiple candidate parameterisations.
- When ORA output is already a final list of significantly enriched pathways with BH FDR-adjusted q-values and you only need to report that list, not compare counts across settings.
- When pathway p-values have not been computed (e.g. input is a raw feature-abundance matrix that has not yet gone through imputation, log2 transformation, auto-scaling, t-testing, and Fisher's exact ORA).

## Inputs

- Candidate lists of differentially abundant (DA) metabolites (e.g. cumulative lists adding one compound at a time in ascending t-test p-value order, or lists thresholded on Bonferroni/BH FDR-adjusted p-values at alpha = 0.005, 0.05, 0.1)
- ORA output: pathway p-values for each candidate list, computed with scipy.stats.fisher_exact (right-tailed) on pathway sets restricted to pathways containing >= 3 compounds
- Five processed metabolomics datasets (e.g. MTBLS135, MTBLS136, Yachida et al. 2019, Quiros et al. 2017, Fuhrer et al. 2017) with imputed missing values (half of each metabolite's minimum), log2-transformed, auto-scaled

## Outputs

- sep_counts_table.csv — per-dataset, per-candidate-list counts of significant enriched pathways (SEPs) at p <= 0.1 without multiple testing correction
- Per-dataset sensitivity curves of SEP count against DA-list construction (e.g. as plot outputs in reproducible_simulations.ipynb)
- Tabulated comparison showing Bonferroni < BH FDR SEP-count ordering, with the BH FDR SEP-count maximum at alpha <= 0.05

## How to apply

After running ORA (right-tailed Fisher's exact / hypergeometric via scipy.stats.fisher_exact, restricted to pathways containing at least 3 compounds and only computing a pathway p-value when k >= 1 DA metabolite maps to it), count the number of significant enriched pathways per candidate DA-metabolite list and per dataset. In this study SEPs were selected at p <= 0.1 without multiple testing correction, while metabolites of interest were generated by BH FDR or Bonferroni adjustment of two-tailed Student's t-tests at alpha thresholds of 0.005, 0.05, and 0.1 (default q <= 0.05). Tabulate the SEP counts by correction method and alpha threshold into sep_counts_table.csv (a per-dataset matrix of pathway counts), and additionally plot per-dataset sensitivity curves of SEP count against DA-list construction. The rationale is that identical background and pathway sets but different DA-list construction yield profoundly different SEP counts — the counts make parameter sensitivity directly comparable across datasets.

## Related tools

- **scipy stats fisher_exact** (Computes right-tailed Fisher's exact (hypergeometric) pathway p-values used as input to the SEP count tabulation)
- **Python 3.8** (Execution environment for the simulations and SEP-count tabulation)
- **Jupyter Notebook** (Hosts reproducible_simulations.ipynb, the notebook in which SEP counts are tabulated into sep_counts_table.csv and plotted)
- **Google Colaboratory** (Alternative remote execution of the same reproducible_simulations.ipynb notebook without local installation) — https://research.google.com/colaboratory/faq.html

## Examples

```
jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- The number of SEPs is a non-negative integer per (dataset, candidate DA list) pair, and equals the count of pathways with Fisher's exact p <= 0.1 (no multiple testing correction).
- Across DA-list sizes from 0 to the full background set, the SEP count must show a rise-then-fall trend: rising to a global maximum then falling to zero once the DA list equals the entire background set.
- For the BH FDR vs Bonferroni comparison at each alpha threshold, the tabulated counts must satisfy Bonferroni SEP count < BH FDR SEP count, with the BH FDR SEP-count maximum occurring at alpha <= 0.05.
- Every candidate DA list must yield the same pathway set input (>= 3 compounds per pathway; p-value computed only when k >= 1) so that SEP-count differences are attributable solely to the DA-list construction, not pathway filtering changes.
- sep_counts_table.csv must contain one row/column per dataset and candidate list combination — verify the table matches the five datasets and the tested alpha thresholds (0.005, 0.05, 0.1).

## Limitations

- No ground-truth dataset is available where the identities of enriched pathways are known, so SEP counts cannot be validated against a known-correct number of pathways.
- In most examples a p-value threshold of P <= 0.1 was used without multiple testing correction to select SEPs, so tabulated counts are sensitive to that unadjusted cutoff.
- The counts reflect SEP totals only — they do not indicate which specific pathways are gained or lost, and single compounds can cause local extrema in the SEP-count curve.

## Evidence

- [results] Count significant enriched pathways (SEPs) at p <= 0.1 without multiple testing correction for each candidate list and dataset.: [evidence span withheld — anchor status: not_found]
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA.: [evidence span withheld — anchor status: not_found]
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
- [discussion] The pathway loss rate represents the proportion of the original pathways (0% misidentification) significant at p <= 0.1 that are no longer significant at f % misidentification.: [evidence span withheld — anchor status: not_found]
- [abstract] changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results: "changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results"
