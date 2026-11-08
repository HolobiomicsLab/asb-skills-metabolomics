---
name: score-normalization-to-unit-range
description: Use when combining annotation scores from multiple MS/MS annotation tools
  with an external additive term (e.g., taxonomic-distance bonuses) whose weights
  were optimized on a 0–1 score scale; without normalization the tools' incomparable
  score ranges dominate or are dominated by the added term.
when_to_use_negative:
- Scores will be used directly for thresholding or reporting per-tool confidence,
  not combined with an external additive term — normalization changes the score meaning
- The candidate scores already share a common, pre-normalized scale across tools
- The analysis targets ubiquitous (primary) metabolites, where taxonomically informed
  re-ranking provides little benefit
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: taxo_scorer
  role: Implements the cleaning/standardization, 0–1 score normalization, taxonomic
    bonus addition and re-ranking pipeline (2_taxo_scorer.Rmd); maintained under the
    tima repository
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained repository containing everything needed to perform Taxonomically
    Informed Metabolite Annotation, including the scoring/normalization workflow
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization used to find the optimal taxo-score weight
    combination applied to the normalized scores (0.81/1.62/2.55)
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: gnfinder
  role: Extracts taxonomic information from free text biological-source fields so
    taxa-level bonuses can be assigned before score combination
  repo: https://github.com/gnames/gnfinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:a486ddb0201b0c80006ae77a073eb11833aaa2a76616ffe16f93127448d7738c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - gnfinder
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# score-normalization-to-unit-range

## Summary

Rescales the heterogeneous native annotation scores produced by computational metabolite annotation tools (ISDB-DNP, Sirius, MS-Finder) to the 0–1 range so they can be combined additively with taxonomic bonus scores (0.81 family, 1.62 genus, 2.55 species) during taxonomically informed re-ranking.

## When to use

Use when combining annotation scores from multiple MS/MS annotation tools with an external additive term (e.g., taxonomic-distance bonuses) whose weights were optimized on a 0–1 score scale; without normalization the tools' incomparable score ranges dominate or are dominated by the added term.

## When NOT to use

- Scores will be used directly for thresholding or reporting per-tool confidence, not combined with an external additive term — normalization changes the score meaning
- The candidate scores already share a common, pre-normalized scale across tools
- The analysis targets ubiquitous (primary) metabolites, where taxonomically informed re-ranking provides little benefit

## Inputs

- Cleaned annotation tool outputs (ISDB-DNP, MS-Finder, Sirius) with per-candidate native scores
- Resolved taxonomic positions of the source organisms (from the 2,107-entry benchmarking dataset, resolved against Catalogue of Life)
- Per-feature candidate structure lists (InChIKey-indexed)

## Outputs

- Normalized (0–1) annotation score column per tool
- Combined taxo-scored candidate rankings per feature
- taxo_scored_rank1_counts.csv with rank-1 correct annotation counts per tool

## How to apply

First clean and standardize each tool's output (the article's 1_taxo_cleaner.Rmd step) so that candidate annotations per feature carry a single numeric score column. Then min-max normalize each tool's original annotation score independently to the 0–1 range, keeping per-tool normalization separate because ISDB-DNP, Sirius and MS-Finder use different native scoring scales. Add the taxonomically informed bonus term to the normalized score, using the optimized weights 0.81, 1.62 and 2.55 for family, genus and species taxa-level matches respectively. Re-rank candidate structures per feature by the combined score and count correct annotations at rank 1 per tool (reported as 1,510 ISDB-DNP, 1,508 Sirius, 546 MS-Finder on the 2,107-spectrum benchmarking dataset after re-ranking). Note the weights are training-set dependent and should not be treated as absolute optima.

## Related tools

- **taxo_scorer** (Implements the cleaning/standardization, 0–1 score normalization, taxonomic bonus addition and re-ranking pipeline (2_taxo_scorer.Rmd); maintained under the tima repository) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained repository containing everything needed to perform Taxonomically Informed Metabolite Annotation, including the scoring/normalization workflow) — https://github.com/taxonomicallyinformedannotation/tima
- **ParBayesianOptimization** (Parallelizable Bayesian optimization used to find the optimal taxo-score weight combination applied to the normalized scores (0.81/1.62/2.55)) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **gnfinder** (Extracts taxonomic information from free text biological-source fields so taxa-level bonuses can be assigned before score combination) — https://github.com/gnames/gnfinder

## Examples

```
Rscript -e "rmarkdown::render('2_taxo_scorer.Rmd')"  # normalize tool scores to 0-1, add taxo weights (0.81/1.62/2.55), re-rank
```

## Evaluation signals

- All normalized score values fall within [0, 1] for each tool before the taxonomic bonus is added
- Rank-1 correct annotation counts after re-ranking approach the reported values (1,510 ISDB-DNP, 1,508 Sirius, 546 MS-Finder on 2,107 spectra)
- Counting benchmark only covers features whose taxonomy resolved against the Catalogue of Life down to species level
- Re-ranked F1 scores show the reported 1.5- to 7-fold systematic improvement over the tools' native ranking

## Limitations

- The optimal weights (0.81, 1.62, 2.55) are heavily dependent on the training sets and should not be taken as absolute optimal values
- Taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules
- Benchmarking favors ISDB-DNP because all benchmarking compounds are present in the Dictionary of Natural Products (v 27.1), and GNPS spectral libraries were part of the Sirius training set, confounding tool comparison

## Evidence

- [full_text] Normalize each tool's original annotation score to the 0-1 range.: [evidence span withheld — anchor status: not_found]
- [discussion] optimal taxo-informed weights: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] taxo scoring workflow scripts: "2) taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
- [discussion] weights are training-set dependent: "Such results should be taken with caution, and not as absolute optimal values, as such optimization process are heavily dependent on the training sets"
- [discussion] cleaning precedes scoring: "1) cleaning and standardization of the outputs (1_taxo_cleaner.Rmd)"
