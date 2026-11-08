---
name: score-percentile-threshold-analysis
description: Use when evaluating whether combining genomic–metabolomic link scoring
  functions (e.g. IOKR, standardised Metcalf/correlation) yields more validated links
  than individual scores, using percentile thresholds and pooled Fisher-style significance
  tests.
when_to_use_negative:
- The analysis goal is de novo BGC–spectrum scoring rather than validating existing
  scores against a set of known validated links
- The data set has no validated-link ground truth labels, since the enrichment proportions
  cannot be computed
- Only a single scoring function is available and no complementary score (e.g. IOKR
  plus standardised strain correlation) exists, making the complementarity test moot
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPLinker
  role: Framework used to integrate genomics and metabolomics data and compute link
    scores (e.g. Metcalf scoring) that feed the percentile-threshold enrichment analysis
  repo: https://github.com/NPLinker/nplinker
- name: nplinker
  role: Python package (installable via pip, ~300MB) providing load_data and get_links
    used to generate per-pair link scores
  repo: https://github.com/NPLinker/nplinker
- name: Python
  role: Scripting environment (version >= 3.11, e.g. via conda create -n npl-3.11
    python=3.11) for computing percentiles, counting validated links and running Fisher-style
    pooled tests
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1008920
    title: Ranking microbial metabolomic and genomic links in the NPLinker framework
      using complementary scoring functions
  source_content_hash: sha256:64459341a2c1d9339c2239a5a17d065f14ceb6b253346a417e6d88b8225c6a99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - NPLinker
  - Python
  - nplinker
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# score-percentile-threshold-analysis

## Summary

Reproduces the 90th-percentile enrichment and complementarity analysis from the NPLinker framework paper, testing whether links above the 90th percentile on both IOKR and standardised strain correlation scores are enriched for validated links relative to either score alone.

## When to use

Use when evaluating whether combining genomic–metabolomic link scoring functions (e.g. IOKR, standardised Metcalf/correlation) yields more validated links than individual scores, using percentile thresholds and pooled Fisher-style significance tests.

## When NOT to use

- The analysis goal is de novo BGC–spectrum scoring rather than validating existing scores against a set of known validated links
- The data set has no validated-link ground truth labels, since the enrichment proportions cannot be computed
- Only a single scoring function is available and no complementary score (e.g. IOKR plus standardised strain correlation) exists, making the complementarity test moot

## Inputs

- Per-pair link score tables for the Crüsemann, Gross and Leão data sets (raw correlation, standardised correlation, IOKR, combined scores)
- Validated-link ground-truth labels for each data set
- GNPS molecular networking data and AntiSMASH BGC data processed via NPLinker

## Outputs

- table2_recomputed.csv with per-dataset and pooled proportions of validated links above the 90th-percentile threshold per scoring method
- Pooled Fisher-style p-values for IOKR, standardised correlation and joint enrichment

## How to apply

Load per-pair link scores and validated-link ground-truth labels for the Crüsemann, Gross and Leão data sets covering raw correlation, standardised correlation, IOKR and combined scores. For each data set and scoring method, compute the 90th percentile of the score distribution as the reporting threshold, then count validated links above that threshold as a proportion of all links (reproducing Table 2). Pool per-dataset counts across the three data sets and run Fisher-style pooled significance tests for IOKR (reported p = 0.0139), standardised correlation (p = 2.483e-11) and joint enrichment of both scores (p = 2.633e-4 and p = 0.0208, starting from IOKR and standardised strain correlation respectively). Write the recomputed proportions and p-values to table2_recomputed.csv and compare against the reported values to judge correctness.

## Related tools

- **NPLinker** (Framework used to integrate genomics and metabolomics data and compute link scores (e.g. Metcalf scoring) that feed the percentile-threshold enrichment analysis) — https://github.com/NPLinker/nplinker
- **nplinker** (Python package (installable via pip, ~300MB) providing load_data and get_links used to generate per-pair link scores) — https://github.com/NPLinker/nplinker
- **Python** (Scripting environment (version >= 3.11, e.g. via conda create -n npl-3.11 python=3.11) for computing percentiles, counting validated links and running Fisher-style pooled tests)

## Examples

```
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Recomputed IOKR enrichment p-value matches the reported 0.0139
- Recomputed standardised correlation enrichment p-value matches the reported 2.483e-11
- Joint enrichment p-values match the reported 2.633e-4 (starting from IOKR) and 0.0208 (starting from standardised strain correlation)
- Per-dataset proportions above the 90th-percentile threshold reproduce Table 2 for the Crüsemann, Gross and Leão data sets
- table2_recomputed.csv is produced and contains one row per data set × scoring method with counts and proportions

## Limitations

- Significance depends on pooling across only three data sets (Crüsemann, Gross, Leão); per-dataset sample sizes can be small
- Results are only interpretable for data sets that include validated links; the paper verified results 'using publicly available data sets that include validated links'
- The discussion section lacks discussion of limitations or reproducibility gaps beyond a changelog, so no additional documented caveats are available

## Evidence

- [results] Links scoring above the 90th percentile on both scores are significantly enriched for validated links compared to links exceeding either individual score alone: [evidence span withheld — anchor status: not_found]
- [methods] metcalf link computation: "# compute the links for the first 3 GCFs using metcalf scoring method
link_graph = npl.get_links(npl.gcfs[:3], "metcalf")"
- [methods] NPLinker framework description: "NPLinker is a python framework for data mining microbial natural products by integrating genomics and metabolomics data."
- [readme] install nplinker: "# install nplinker package (requiring ~300MB of disk space)
pip install nplinker"
