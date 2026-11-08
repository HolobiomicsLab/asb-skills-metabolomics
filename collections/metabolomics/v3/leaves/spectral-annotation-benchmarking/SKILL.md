---
name: spectral-annotation-benchmarking
description: Use when you need to quantitatively benchmark MS/MS annotation tools
  (ISDB-DNP, MS-Finder, Sirius) and tune taxonomically informed re-ranking weights
  on a labeled benchmark set of identified molecules.
when_to_use_negative:
- Do not treat the optimized 0.81/1.62/2.55 combination as absolute optimal values
  on new training sets — the optimization is heavily dependent on the training set
  used.
- Do not apply taxonomically informed re-ranking to ubiquitous molecules; the benefit
  is concentrated in specialized metabolites.
- Do not use this benchmark to fairly compare ISDB-DNP against other tools when all
  benchmark compounds are in the DNP, since ISDB-DNP is favored by construction.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization of the three taxonomically informed score
    parameters across degraded blocks
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: taxo_scorer
  role: Computes taxonomically informed re-ranking scores weighted at family, genus,
    and species taxa levels (2_taxo_scorer.Rmd)
  repo: https://github.com/oolonek/taxo_scorer
- name: gnfinder
  role: Extracts taxonomic names from free-text biological source fields for resolution
    against the Global Names index / Catalogue of Life
  repo: https://github.com/gnames/gnfinder
- name: ISDB-DNP
  role: In silico spectral annotation tool whose candidate lists are benchmarked and
    re-ranked
  repo: http://oolonek.github.io/ISDB/
- name: tima
  role: Maintained repository containing the taxonomically informed metabolite annotation
    pipeline including the cleaner and scorer steps
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:2a704a31f1f7b3df04a1cd92c9b675b7143c192f66e2977e5237b0bd38c751a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AnotherSamWilson/ParBayesianOptimization
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - gnfinder
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spectral-annotation-benchmarking

## Summary

Benchmark and tune taxonomically informed metabolite annotation by scoring ISDB-DNP candidate re-rankings against a 2,107-entry benchmarking dataset, using ParBayesianOptimization to find optimal family/genus/species scores (0.81/1.62/2.55).

## When to use

Use when you need to quantitatively benchmark MS/MS annotation tools (ISDB-DNP, MS-Finder, Sirius) and tune taxonomically informed re-ranking weights on a labeled benchmark set of identified molecules.

## When NOT to use

- Do not treat the optimized 0.81/1.62/2.55 combination as absolute optimal values on new training sets — the optimization is heavily dependent on the training set used.
- Do not apply taxonomically informed re-ranking to ubiquitous molecules; the benefit is concentrated in specialized metabolites.
- Do not use this benchmark to fairly compare ISDB-DNP against other tools when all benchmark compounds are in the DNP, since ISDB-DNP is favored by construction.

## Inputs

- ISDB-DNP annotation outputs (.mgf spectral files and annotation tables) from OSF deposit osf.io/bvs6x
- 2,107-entry benchmarking dataset of identified molecules
- Biological source / taxonomic metadata (species, genus, family) for annotations and candidates
- Catalogue of Life taxonomy backbone for name resolution

## Outputs

- optimal_scores.csv with averaged block-wise optimal family/genus/species weights
- Block-wise Bayesian optimization parameter combinations (four degraded blocks)
- F1 score comparison of annotation tools with and without taxonomically informed re-ranking

## How to apply

Retrieve the ISDB-DNP annotation outputs and the 2,107-entry benchmarking dataset from the OSF deposit (osf.io/bvs6x). Clean and standardize the tool outputs with 1_taxo_cleaner.Rmd, extracting taxonomic information from free-text biological source fields using gnfinder and resolving names against the Catalogue of Life. To benchmark under degraded metadata, randomly split the annotations into four independent blocks with incomplete taxonomy, then compute re-ranking scores with 2_taxo_scorer.Rmd, weighting similarity contributions at the family, genus, and species taxa levels. Tune the three score parameters with the ParBayesianOptimization package in parallel mode on each degraded block (the package alternates Gaussian-process fitting and acquisition-function maximization over the parameter space), then average the block-wise optima. Validate by comparing the averaged weights against the reported optimum 0.81 (family), 1.62 (genus), 2.55 (species) — scores must increase with lower (finer) taxa levels — and assess F1-score improvement, which should reach 1.5- to 7-fold over the un-reranked tool.

## Related tools

- **ParBayesianOptimization** (Parallelizable Bayesian optimization of the three taxonomically informed score parameters across degraded blocks) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **taxo_scorer** (Computes taxonomically informed re-ranking scores weighted at family, genus, and species taxa levels (2_taxo_scorer.Rmd)) — https://github.com/oolonek/taxo_scorer
- **gnfinder** (Extracts taxonomic names from free-text biological source fields for resolution against the Global Names index / Catalogue of Life) — https://github.com/gnames/gnfinder
- **ISDB-DNP** (In silico spectral annotation tool whose candidate lists are benchmarked and re-ranked) — http://oolonek.github.io/ISDB/
- **tima** (Maintained repository containing the taxonomically informed metabolite annotation pipeline including the cleaner and scorer steps) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
install.packages("ParBayesianOptimization"); devtools::install_github("AnotherSamWilson/ParBayesianOptimization")
```

## Evaluation signals

- The averaged block-wise optima must reproduce 0.81 (family), 1.62 (genus), and 2.55 (species).
- Optimal scores must be inversely proportional to taxa hierarchical position: family < genus < species.
- The benchmark must contain 2,107 distinct entries, and taxonomic degradation must be run as four independent blocks.
- Taxonomically informed scoring must yield a systematic F1 score improvement of 1.5- to 7-fold over the un-reranked annotation tool.
- Benchmark entries retained must have taxonomy resolved against the Catalogue of Life and complete down to species level.

## Limitations

- The optimized score combination is heavily dependent on the training set and should not be taken as absolute optimal values.
- All compounds of the benchmarking dataset are in the DNP, so ISDB-DNP is favored and tool comparisons are not fully fair; GNPS spectral libraries were also part of the Sirius training set.
- 197 benchmarking entries show large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; these are listed in Problematic_entries.csv but cannot be identified by annotation tools.
- Taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules.

## Evidence

- [discussion] the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively): "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] A parallelizable Bayesian optimization algorithm ... was then used: "A parallelizable Bayesian optimization algorithm (https://github.com/AnotherSamWilson/ParBayesianOptimization) was then used"
- [discussion] constituted by 2107 distinct entries and was named benchmarking dataset: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] Such results should be taken with caution, and not as absolute optimal values: "Such results should be taken with caution, and not as absolute optimal values, as such optimization process are heavily dependent on the training sets"
- [readme] Script to proceed to taxonomically informed scoring in metabolite annotation: [evidence span withheld — anchor status: external_doc]
