---
name: taxonomic-distance-scoring
description: Use when you have ranked candidate annotation lists (e.g. from ISDB-DNP,
  Sirius or MS-Finder) plus resolved taxonomy of the producing organism, and want
  to re-rank candidates using taxonomic distance to improve precision, recall and
  F1.
when_to_use_negative:
- The molecules of interest are ubiquitous (primary) metabolites — taxonomic scoring
  mostly benefits specialized metabolites, not ubiquitous molecules
- Candidate annotations come from a tool whose spectral scores should remain independent
  and you only want hard taxonomic filtering — the paper argues taxonomically informed
  scoring conserving score independence is preferable to basic filtering, so filtering
  is a different skill
- Organism taxonomy cannot be resolved down to the species level against the taxonomy
  backbone (e.g. Catalogue of Life)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: taxo_scorer
  role: R workflow (2_taxo_scorer.Rmd) that computes the taxonomically informed scores
    and re-ranks candidates; maintained at github.com/taxonomicallyinformedannotation/
  repo: https://github.com/oolonek/taxo_scorer
- name: gnfinder
  role: Extracts taxonomic names from free text source metadata and matches them against
    the Global Names index for resolution
  repo: https://github.com/gnames/gnfinder
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization of the family/genus/species score weights
    over degraded-taxonomy blocks
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: tima
  role: R package implementing the full Taxonomically Informed Metabolite Annotation
    pipeline, installable via docker adafede/tima-r
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: ISDB-DNP
  role: In silico annotation tool whose output candidate lists are re-ranked; benchmarked
    with 7-fold F1 increase and optimal weights 0.81/1.62/2.55
  repo: http://oolonek.github.io/ISDB/
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:20dd5388c70e884e59f65592b6e92a999157f67598865808ffee59e8c467739b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - Sirius
  - gnfinder
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# taxonomic-distance-scoring

## Summary

Re-ranks candidate metabolite annotations by complementing each tool's normalized spectral score with taxonomic-similarity contributions at the family, genus and species taxa levels, so that candidates from organisms taxonomically close to the biological source rank higher.

## When to use

Use when you have ranked candidate annotation lists (e.g. from ISDB-DNP, Sirius or MS-Finder) plus resolved taxonomy of the producing organism, and want to re-rank candidates using taxonomic distance to improve precision, recall and F1.

## When NOT to use

- The molecules of interest are ubiquitous (primary) metabolites — taxonomic scoring mostly benefits specialized metabolites, not ubiquitous molecules
- Candidate annotations come from a tool whose spectral scores should remain independent and you only want hard taxonomic filtering — the paper argues taxonomically informed scoring conserving score independence is preferable to basic filtering, so filtering is a different skill
- Organism taxonomy cannot be resolved down to the species level against the taxonomy backbone (e.g. Catalogue of Life)

## Inputs

- Ranked candidate annotation lists from ISDB-DNP, Sirius and MS-Finder for the 2,107-spectrum benchmarking entries (OSF deposit osf.io/bvs6x)
- Resolved taxonomic positions of the organisms (species, genus, family), matched against Catalogue of Life
- Normalized (0-1) original annotation scores per tool

## Outputs

- Re-ranked candidate lists with combined spectral + taxonomic scores per feature
- Per-tool before/after precision, recall, F1 and rank-1 correct-annotation counts (benchmark_metrics.csv, taxo_scored_rank1_counts.csv)
- Optimized score parameters (optimal_scores.csv): 0.81 / 1.62 / 2.55 for family / genus / species

## How to apply

Clean and standardize the annotation tool outputs (1_taxo_cleaner.Rmd) and resolve the organism's taxonomic position against a backbone such as Catalogue of Life, extracting names from free text with gnfinder if needed. Normalize each tool's original annotation score to the 0-1 range, then apply the taxonomically informed scoring (2_taxo_scorer.Rmd) by adding weighted similarity contributions at the family, genus and species taxa levels; the Bayesian-optimized combination reported for ISDB-DNP is 0.81 (family), 1.62 (genus) and 2.55 (species), i.e. inversely proportional to taxa hierarchical position. Re-rank candidate structures per feature by the combined score and recompute TP (correct structure at rank 1), FP and FN to derive precision, recall and F1 before and after re-ranking. On the 2,107-spectrum benchmarking dataset this raised rank-1 correct annotations to 1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder), giving 7-fold, 1.5-fold and 3-fold F1 increases respectively. Optimize the three weights with ParBayesianOptimization in parallel over degraded-taxonomy blocks if re-tuning, and average block-wise optima.

## Related tools

- **taxo_scorer** (R workflow (2_taxo_scorer.Rmd) that computes the taxonomically informed scores and re-ranks candidates; maintained at github.com/taxonomicallyinformedannotation/) — https://github.com/oolonek/taxo_scorer
- **gnfinder** (Extracts taxonomic names from free text source metadata and matches them against the Global Names index for resolution) — https://github.com/gnames/gnfinder
- **ParBayesianOptimization** (Parallelizable Bayesian optimization of the family/genus/species score weights over degraded-taxonomy blocks) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **tima** (R package implementing the full Taxonomically Informed Metabolite Annotation pipeline, installable via docker adafede/tima-r) — https://github.com/taxonomicallyinformedannotation/tima
- **ISDB-DNP** (In silico annotation tool whose output candidate lists are re-ranked; benchmarked with 7-fold F1 increase and optimal weights 0.81/1.62/2.55) — http://oolonek.github.io/ISDB/

## Examples

```
Rscript -e "rmarkdown::render('2_taxo_scorer.Rmd', params = list(score_family = 0.81, score_genus = 1.62, score_species = 2.55))"
```

## Evaluation signals

- Reproduced F1 fold-increases (post/pre) per tool match the reported 7-fold (ISDB-DNP), 1.5-fold (Sirius) and 3-fold (MS-Finder)
- Rank-1 correct annotation counts after re-ranking reach ~1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder) on the 2,107-entry benchmark
- Averaged Bayesian-optimized weights reproduce 0.81 (family), 1.62 (genus), 2.55 (species), confirming scores inversely proportional to taxa hierarchical position
- Recomputed TP/FP/FN on re-ranked lists show systematic increase in both precision and recall relative to the baseline rank-1 evaluation
- All before/after metrics written to benchmark_metrics.csv are finite and per-tool TP + FN equals the number of correct reference structures

## Limitations

- The optimized score combination is heavily dependent on the training sets and should not be taken as absolute optimal values
- Taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules
- All compounds of the benchmarking dataset are present in the DNP, so ISDB-DNP is favored and tool performance comparison is not fair; GNPS spectral libraries were also part of the Sirius training set
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the list is in Problematic_entries.csv but tools cannot identify such erroneous entries

## Evidence

- [discussion] optimal scores: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] F1 increase: "was shown to systematically improve the F1 score of the evaluated solutions (ISDB-DNP, Sirius, MS-Finder) with a 1.5 to 7-fold increase"
- [intro] scoring better than filtering: "a taxonomically informed scoring allowing, both, to consider spectral similarity and taxonomic information while conserving the independence of the individual resulting scores appears as a better"
- [discussion] workflow scripts: "2) taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
- [readme] readme pointer: [evidence span withheld — anchor status: external_doc]
