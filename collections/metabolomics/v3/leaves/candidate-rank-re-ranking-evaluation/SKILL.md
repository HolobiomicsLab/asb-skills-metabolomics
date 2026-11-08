---
name: candidate-rank-re-ranking-evaluation
description: Use when you have ranked candidate structure lists from tools like ISDB-DNP,
  Sirius or MS-Finder plus organism taxonomy, and want to quantify how taxonomically
  informed re-ranking changes rank-1 annotation correctness.
when_to_use_negative:
- The target metabolites are ubiquitous rather than specialized — taxonomically informed
  scoring mostly benefits specialized metabolites
- Candidate entries lack biological source information resolved against the Catalogue
  of Life down to the species level
- The optimized score weights are transferred verbatim to a new training set — the
  article warns such values are heavily dependent on the training sets and should
  not be taken as absolute optima
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: taxo_scorer
  role: Script performing taxonomically informed scoring and re-ranking of candidate
    annotation lists
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained repository containing everything needed to perform Taxonomically
    Informed Metabolite Annotation, including the scoring/re-ranking step (2_taxo_scorer.Rmd)
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization used to find the optimal taxo-informed
    score combination (0.81 family, 1.62 genus, 2.55 species)
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:165022ba5380fb0dd6431b85d6b20c088c694391f7bc527859cfac9199ccc39a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - Sirius
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# candidate-rank-re-ranking-evaluation

## Summary

Re-ranks computational metabolite annotation candidate lists using taxonomically informed scoring (taxonomic distance between the producing organism and candidate structures) and evaluates the impact on precision, recall and F1 with correct-at-rank-1 as true positives.

## When to use

Use when you have ranked candidate structure lists from tools like ISDB-DNP, Sirius or MS-Finder plus organism taxonomy, and want to quantify how taxonomically informed re-ranking changes rank-1 annotation correctness.

## When NOT to use

- The target metabolites are ubiquitous rather than specialized — taxonomically informed scoring mostly benefits specialized metabolites
- Candidate entries lack biological source information resolved against the Catalogue of Life down to the species level
- The optimized score weights are transferred verbatim to a new training set — the article warns such values are heavily dependent on the training sets and should not be taken as absolute optima

## Inputs

- Ranked candidate annotation lists (output of ISDB-DNP, Sirius and/or MS-Finder) for each benchmarking entry
- Benchmarking dataset input/output data from the OSF deposit (https://osf.io/bvs6x/)
- Taxonomic information of the biological source (species, genus, family) resolved against Catalogue of Life

## Outputs

- benchmark_metrics.csv with before/after TP, FP, FN, precision, recall, F1 and per-tool F1 fold-increase
- Re-ranked candidate lists per tool after taxonomically informed scoring

## How to apply

Parse the ranked candidate annotation lists for each entry (e.g. the 2,107-spectrum benchmarking dataset from OSF https://osf.io/bvs6x/), compute baseline TP (correct structure at rank 1), FP (wrong structure at rank 1) and FN (correct structure at rank >1), and derive precision, recall and F1. Apply taxonomically informed re-ranking with the optimized score combination — 0.81 (family), 1.62 (genus) and 2.55 (species) for ISDB-DNP — to re-order candidates per tool, then recompute TP/FP/FN, precision, recall and F1 on the re-ranked lists. Compute the per-tool F1 fold-increase (post/pre) and compare against reported increases: 7-fold for ISDB-DNP, 1.5-fold for Sirius and 3-fold for MS-Finder, with rank-1 correct annotations rising to 1,510, 1,508 and 546 respectively. Score weights can be tuned with the parallelizable Bayesian optimization algorithm (ParBayesianOptimization), but the article cautions they are training-set dependent. Write all before/after metrics and fold-increases to benchmark_metrics.csv.

## Related tools

- **taxo_scorer** (Script performing taxonomically informed scoring and re-ranking of candidate annotation lists) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained repository containing everything needed to perform Taxonomically Informed Metabolite Annotation, including the scoring/re-ranking step (2_taxo_scorer.Rmd)) — https://github.com/taxonomicallyinformedannotation/tima
- **ParBayesianOptimization** (Parallelizable Bayesian optimization used to find the optimal taxo-informed score combination (0.81 family, 1.62 genus, 2.55 species)) — https://github.com/AnotherSamWilson/ParBayesianOptimization

## Examples

```
Rscript -e "rmarkdown::render('2_taxo_scorer.Rmd')"
```

## Evaluation signals

- Per-tool F1 fold-increase (post/pre re-ranking) reproduces the reported 7-fold (ISDB-DNP), 1.5-fold (Sirius) and 3-fold (MS-Finder) increases
- Rank-1 correct annotation counts after re-ranking match the reported 1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder)
- Baseline TP/FP/FN are computed with correct-at-rank-1 as TP, wrong-at-rank-1 as FP and correct-at-rank->1 as FN before any re-ranking
- benchmark_metrics.csv contains before/after precision, recall and F1 for every benchmarked tool
- Recall and precision both increase systematically after re-ranking, not just one of the two

## Limitations

- The optimized score combination (0.81, 1.62, 2.55 for family/genus/species) is heavily dependent on the training sets and should not be taken as absolute optimal values
- The benchmarking does not allow a fair comparison of tool performances because all compounds are present in the DNP, favoring ISDB-DNP
- GNPS spectral libraries were part of the Sirius training set, confounding tool comparison
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the list is provided as Problematic_entries.csv but erroneous entries cannot be identified by annotation tools
- Taxonomically informed scoring will mostly benefit annotation of specialized metabolites and not ubiquitous molecules

## Evidence

- [discussion] Optimized score weights: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] Systematic F1 improvement: "was shown to systematically improve the F1 score of the evaluated solutions (ISDB-DNP, Sirius, MS-Finder) with a 1.5 to 7-fold increase"
- [discussion] Re-ranking step script: "2) taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
- [readme] taxo_scorer README: [evidence span withheld — anchor status: external_doc]
- [discussion] Tool comparison caveat: "all compounds of the benchmarking dataset are present in the DNP, and the ISDB-DNP tool, which is by definition backed by the same DB is thus favored"
