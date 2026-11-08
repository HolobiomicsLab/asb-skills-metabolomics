---
name: bayesian-hyperparameter-optimization
description: Use when re-ranking ISDB-DNP (or similar MS annotation) candidate structures
  with taxonomically informed scores and the family/genus/species weight combination
  must be fitted rather than hand-set.
when_to_use_negative:
- The taxonomic weight combination is already known and validated (0.81 / 1.62 / 2.55)
  and only needs to be applied by 2_taxo_scorer.Rmd — re-optimization is unnecessary
- The scoring system is a plain taxonomic filtering process rather than a scoring
  objective with tunable continuous weights that a Gaussian-process optimizer can
  explore
- Benchmarking compounds are not present in the reference database (e.g. non-DNP annotations),
  making the F1 objective on the benchmarking dataset an unreliable optimization target
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization algorithm used to tune the family, genus
    and species score parameters over each degraded block
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: taxo_scorer
  role: Provides the taxonomically informed scoring and re-ranking objective (2_taxo_scorer.Rmd)
    whose parameters are optimized
  repo: https://github.com/oolonek/taxo_scorer
- name: ISDB-DNP
  role: Source annotation set whose candidate structures are re-scored and whose optimized
    weight combination is derived
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:1b7d4b4a662494d9496d33707b50e13b1da492a55cbb683516091bda86537f0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AnotherSamWilson/ParBayesianOptimization
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - taxo_scorer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bayesian-hyperparameter-optimization

## Summary

Tunes the taxonomically informed scoring parameters (family, genus, species weights) used to re-rank metabolite annotation candidates by optimizing the taxo_scorer objective with the parallelizable ParBayesianOptimization R package, converging on scores of 0.81, 1.62 and 2.55.

## When to use

Use when re-ranking ISDB-DNP (or similar MS annotation) candidate structures with taxonomically informed scores and the family/genus/species weight combination must be fitted rather than hand-set.

## When NOT to use

- The taxonomic weight combination is already known and validated (0.81 / 1.62 / 2.55) and only needs to be applied by 2_taxo_scorer.Rmd — re-optimization is unnecessary
- The scoring system is a plain taxonomic filtering process rather than a scoring objective with tunable continuous weights that a Gaussian-process optimizer can explore
- Benchmarking compounds are not present in the reference database (e.g. non-DNP annotations), making the F1 objective on the benchmarking dataset an unreliable optimization target

## Inputs

- ISDB-DNP annotation output tables (from OSF deposit osf.io/bvs6x)
- Benchmarking dataset of 2,107 distinct entries with taxonomic metadata
- Taxonomically degraded annotation blocks (four independent blocks simulating incomplete taxonomic knowledge)
- Scoring function wrapper exposing the three parameters (family, genus, species taxa-level weights) and returning an F1-style score

## Outputs

- Block-wise optimal parameter combinations from ParBayesianOptimization runs
- Averaged optimal taxo-informed scores written to optimal_scores.csv (expected ~0.81 family, 1.62 genus, 2.55 species)
- Gaussian-process posterior traces / convergence logs from bayesOpt runs

## How to apply

Retrieve the ISDB-DNP annotation outputs and the 2,107-entry benchmarking dataset from the OSF deposit (osf.io/bvs6x). Randomly degrade the taxonomic metadata of the annotations into four independent blocks to simulate incomplete taxonomic knowledge. Compute taxonomically informed re-ranking scores with the taxo_scorer workflow (2_taxo_scorer.Rmd), weighting similarity contributions at the family, genus, and species taxa levels, and define an objective (e.g. F1 score against the benchmark) for the optimizer. Tune the three score parameters with the ParBayesianOptimization package in parallel mode: initial parameters are scored, a Gaussian process is fit, a parameter maximizing an acquisition function is selected and scored, and steps repeat until a stopping criterion is met. Average the block-wise optimal parameter combinations across the four degradation blocks and compare against the reported optima of 0.81 (family), 1.62 (genus) and 2.55 (species).

## Related tools

- **ParBayesianOptimization** (Parallelizable Bayesian optimization algorithm used to tune the family, genus and species score parameters over each degraded block) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **taxo_scorer** (Provides the taxonomically informed scoring and re-ranking objective (2_taxo_scorer.Rmd) whose parameters are optimized) — https://github.com/oolonek/taxo_scorer
- **ISDB-DNP** (Source annotation set whose candidate structures are re-scored and whose optimized weight combination is derived)

## Examples

```
install.packages("ParBayesianOptimization"); library(ParBayesianOptimization); opt <- bayesOpt(FUN = taxo_score, bounds = list(score_family = c(0, 3), score_genus = c(0, 3), score_species = c(0, 3)))
```

## Evaluation signals

- Averaged block-wise optima must reproduce the reported combination of 0.81 (family), 1.62 (genus) and 2.55 (species) and be written to optimal_scores.csv
- Optimized scores must be inversely proportional to taxa hierarchical position (family < genus < species), confirming hierarchical weighting direction
- Each of the four independently degraded blocks must converge on comparable parameter combinations before averaging
- The resulting scores must improve F1 (recall and precision) of ISDB-DNP relative to unweighted annotation scores, consistent with the reported 1.5- to 7-fold increase

## Limitations

- The optimized score combination is dependent on the training set and should not be taken as absolute optimal values
- The optimization was performed with taxonomic metadata deliberately degraded, so optima may shift with complete taxonomic knowledge
- A number of benchmarking entries (197) have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass, adding noise to the F1 objective

## Evidence

- [discussion] A parallelizable Bayesian optimization algorithm (https://github.com/AnotherSamWilson/ParBayesianOptimization) was then used: "A parallelizable Bayesian optimization algorithm (https://github.com/AnotherSamWilson/ParBayesianOptimization) was then used"
- [discussion] Optimal taxo-informed scores for ISDB-DNP: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] Dependence of optima on training sets: "Such results should be taken with caution, and not as absolute optimal values, as such optimization process are heavily dependent on the training sets"
- [readme] ParBayesianOptimization package process: "1.  Initial parameters are scored
2.  Gaussian Process is fit/updated  
3.  Parameter is found which maximizes an acquisition function  
4.  This parameter is scored"
- [discussion] Input/output data availability: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
