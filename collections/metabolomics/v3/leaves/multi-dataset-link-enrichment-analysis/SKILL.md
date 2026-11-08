---
name: multi-dataset-link-enrichment-analysis
description: Use when evaluating whether combining link-scoring functions (e.g. IOKR
  and standardised Metcalf-style strain correlation) yields a higher proportion of
  validated links than individual scores, using percentile thresholds over per-pair
  link scores.
when_to_use_negative:
- When no ground-truth validated-link labels exist for the data sets, since enrichment
  proportions cannot be computed
- When the goal is ranking or generating links themselves (use NPLinker's get_links
  scoring) rather than statistically evaluating score enrichment
- When scores are not comparable on a common percentile basis across data sets, since
  the 90th-percentile threshold is defined per data set per scoring method
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: nplinker
  role: Python framework used to compute the metabolomic–genomic link scores (e.g.
    metcalf-style links) that feed the enrichment analysis
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Used (Python ≥3.11) to compute percentile thresholds, pooled significance
    tests, and write table2_recomputed.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:a99d8273573243de717c3162f8e3e18ff74974254e7a99b453c94f6fb35f1310
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - NPLinker
  - Python
  - nplinker
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# multi-dataset-link-enrichment-analysis

## Summary

Reproduces the 90th-percentile enrichment analysis showing that metabolomic–genomic links scoring above the 90th percentile on both IOKR and standardised strain correlation scores are significantly enriched for validated links compared to either score alone across the Crüsemann, Gross, and Leão data sets (Table 2).

## When to use

Use when evaluating whether combining link-scoring functions (e.g. IOKR and standardised Metcalf-style strain correlation) yields a higher proportion of validated links than individual scores, using percentile thresholds over per-pair link scores.

## When NOT to use

- When no ground-truth validated-link labels exist for the data sets, since enrichment proportions cannot be computed
- When the goal is ranking or generating links themselves (use NPLinker's get_links scoring) rather than statistically evaluating score enrichment
- When scores are not comparable on a common percentile basis across data sets, since the 90th-percentile threshold is defined per data set per scoring method

## Inputs

- Per-pair metabolomic–genomic link scores (raw correlation, standardised correlation, IOKR, combined scores) for the Crüsemann, Gross, and Leão data sets
- Validated-link ground-truth labels for each data set
- NPLinker link graphs (GCF–spectrum links, e.g. from metcalf scoring)

## Outputs

- table2_recomputed.csv with per-dataset proportions of validated links above the 90th-percentile threshold per scoring method
- Pooled Fisher-style p-values for IOKR, standardised correlation, and joint (both-scores) enrichment
- Comparison of recomputed values against reported values (IOKR p = 0.0139; standardised correlation p = 2.483e-11; joint p = 2.633e-4 and 0.0208)

## How to apply

Load per-pair link scores and validated-link ground-truth labels for the Crüsemann, Gross, and Leão data sets covering raw correlation, standardised correlation, IOKR, and combined scores. For each data set and scoring method, compute the 90th percentile of the score distribution as the reporting threshold, then count validated links above the threshold as a proportion of all links to reproduce Table 2. Pool the per-dataset results across the three data sets and run Fisher-style pooled significance tests for IOKR, standardised correlation, and the joint enrichment of both scores. Compare reproduced proportions and p-values against the reported values — IOKR p = 0.0139; standardised correlation p = 2.483e-11; joint enrichment p = 2.633e-4 (from IOKR) and p = 0.0208 (from standardised strain correlation) — and write the full table to table2_recomputed.csv.

## Related tools

- **nplinker** (Python framework used to compute the metabolomic–genomic link scores (e.g. metcalf-style links) that feed the enrichment analysis) — https://github.com/NPLinker/nplinker
- **Python** (Used (Python ≥3.11) to compute percentile thresholds, pooled significance tests, and write table2_recomputed.csv)

## Examples

```
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Recomputed joint-enrichment p-values match the reported values: p = 2.633e-4 (starting from IOKR) and p = 0.0208 (starting from standardised strain correlation)
- Individual-score pooled p-values match reported values: IOKR p = 0.0139 and standardised correlation p = 2.483e-11
- Each scoring method's threshold is exactly the 90th percentile of that method's score distribution within each of the three data sets
- Validated-link proportions reproduce Table 2 for raw correlation, standardised correlation, IOKR, and combined scores
- Output file table2_recomputed.csv contains the full per-dataset and pooled results

## Limitations

- The enrichment result is verified only on publicly available data sets that include validated links (Crüsemann, Gross, Leão); it may not transfer to data sets without validated links
- Significance relies on Fisher-style pooled tests across data sets, so per-dataset heterogeneity is not separately modelled
- The article's discussion section contains no discussion of limitations, missing artifacts, or reproducibility gaps (only a changelog), so failure modes beyond those stated are not documented

## Evidence

- [abstract] We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others.: "We demonstrate that using multiple link-scoring functions together makes it easier to prioritise true links relative to others."
- [abstract] Results verified on public data sets with validated links: "Results are verified using publicly available data sets that include validated links."
- [methods] Link computation via NPLinker get_links: "link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [methods] Python version requirement: "Python version ≥3.11"
