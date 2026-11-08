---
name: candidate-structure-rank-evaluation
description: Use when annotation outputs (ISDB-DNP, Sirius, MS-Finder candidate lists)
  must be re-ranked using biological-source taxonomy of the producing organism, and
  rank-1 annotation accuracy on a benchmarking dataset must be quantified.
when_to_use_negative:
- 'Ubiquitous (primary-type) molecules: taxonomically informed scoring mostly benefits
  specialized metabolites, not ubiquitous molecules.'
- Benchmarks where every compound is present in the Dictionary of Natural Products
  and ISDB-DNP-backed candidates are evaluated — the tool is favored and rank counts
  are not a fair cross-tool comparison.
- 'Situations where a strict taxonomic filter is required rather than soft scoring:
  the paper argues a scoring scheme that conserves the independence of individual
  scores is preferable, but it does not remove out-of-taxonomy candidates.'
tools:
- name: taxo_scorer
  role: Script implementing taxonomically informed scoring and re-ranking of candidate
    structures
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained repository containing everything needed to perform Taxonomically
    Informed Metabolite Annotation, including scoring and re-ranking
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization used to tune the optimal taxonomic score
    weights (0.81, 1.62, 2.55)
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: ISDB-DNP
  role: Source of candidate structures and original annotation scores being re-ranked
- name: SIRIUS
  role: Annotation tool whose output candidates and scores feed the re-ranking
- name: MS-Finder
  role: Annotation tool whose output candidates and scores feed the re-ranking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:a1de11a40c9efdd5f9ea440733f1b719f666f0b611031fe0f4aa95f87e900516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - ParBayesianOptimization
  - SIRIUS
  - taxo_scorer
  - tima
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# candidate-structure-rank-evaluation

## Summary

Re-ranks candidate metabolite structures per spectral feature by combining each annotation tool's normalized score with a taxonomically informed score (0.81 at family, 1.62 at genus, 2.55 at species level), then counts rank-1 correct annotations to quantify annotation confidence gains.

## When to use

Use when annotation outputs (ISDB-DNP, Sirius, MS-Finder candidate lists) must be re-ranked using biological-source taxonomy of the producing organism, and rank-1 annotation accuracy on a benchmarking dataset must be quantified.

## When NOT to use

- Ubiquitous (primary-type) molecules: taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules.
- Benchmarks where every compound is present in the Dictionary of Natural Products and ISDB-DNP-backed candidates are evaluated — the tool is favored and rank counts are not a fair cross-tool comparison.
- Situations where a strict taxonomic filter is required rather than soft scoring: the paper argues a scoring scheme that conserves the independence of individual scores is preferable, but it does not remove out-of-taxonomy candidates.

## Inputs

- Cleaned, standardized annotation outputs from ISDB-DNP, MS-Finder and Sirius (candidate structures with scores per spectral feature)
- Resolved taxonomic positions of the biological source organisms (species/genus/family) for the 2,107-spectrum benchmarking dataset

## Outputs

- taxo_scored_rank1_counts.csv with per-tool counts of correct annotations at rank 1
- Re-ranked candidate annotation tables per tool

## How to apply

Load the cleaned annotation tool outputs for the three tools (ISDB-DNP, MS-Finder, Sirius) alongside the taxonomic positions of the source organisms resolved against Catalogue of Life. Normalize each tool's original annotation score to the 0-1 range so scores remain comparable and independent. Add the taxonomically informed score with the reported optimal weights — 0.81 at family, 1.62 at genus and 2.55 at species level — to the normalized tool score per candidate structure. Re-rank candidate structures per feature by this combined score, then count rank-1 correct annotations per tool and save the result as taxo_scored_rank1_counts.csv. The rationale is that specialized metabolites are strongly linked to the taxonomic position of the producing organism, so taxonomic proximity boosts candidates from related organisms.

## Related tools

- **taxo_scorer** (Script implementing taxonomically informed scoring and re-ranking of candidate structures) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained repository containing everything needed to perform Taxonomically Informed Metabolite Annotation, including scoring and re-ranking) — https://github.com/taxonomicallyinformedannotation/tima
- **ParBayesianOptimization** (Parallelizable Bayesian optimization used to tune the optimal taxonomic score weights (0.81, 1.62, 2.55)) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **ISDB-DNP** (Source of candidate structures and original annotation scores being re-ranked)
- **SIRIUS** (Annotation tool whose output candidates and scores feed the re-ranking)
- **MS-Finder** (Annotation tool whose output candidates and scores feed the re-ranking)

## Examples

```
docker pull adafede/tima-r
```

## Evaluation signals

- After taxo-informed re-ranking on the 2,107-spectrum benchmarking dataset, rank-1 correct annotations should reach 1,510 for ISDB-DNP, 1,508 for Sirius and 546 for MS-Finder.
- F1 score of each evaluated solution should improve systematically relative to the spectral-only baseline, with a reported 1.5- to 7-fold increase.
- Normalization check: every candidate score fed into the combination must lie in the 0-1 range before adding taxonomic increments.
- Weight sanity check: taxonomic increments applied are exactly 0.81 (family), 1.62 (genus) and 2.55 (species) for ISDB-DNP-optimized weights.
- Diff/comparison: re-ranked output should preserve candidate identity per feature while only changing rank order; counts file taxo_scored_rank1_counts.csv must exist with one row per tool.

## Limitations

- The optimized score combination (0.81/1.62/2.55) is heavily dependent on the training sets and should be taken with caution, not as absolute optimal values.
- All compounds of the benchmarking dataset are present in the DNP, so ISDB-DNP is favored by definition and rank counts do not allow a fair comparison of tool performances.
- GNPS spectral libraries were part of the Sirius training set, confounding cross-tool comparison.
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the problematic list (Problematic_entries.csv) is provided but annotation tools cannot identify erroneous entries themselves.

## Evidence

- [discussion] the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively): "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] taxo scoring systematic improvement: "was shown to systematically improve the F1 score of the evaluated solutions (ISDB-DNP, Sirius, MS-Finder) with a 1.5 to 7-fold increase"
- [abstract] abstract re-ranking claim: "the consideration of the taxonomic distance allowed an efficient re-ranking of the candidate structures leading to a systematic enhancement of the recall and precision rates of the tools"
- [discussion] benchmark dataset definition: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [methods] tima repo scope: "This repository contains everything needed to perform **T**axonomically **I**nformed **M**etabolite **A**nnotation."
