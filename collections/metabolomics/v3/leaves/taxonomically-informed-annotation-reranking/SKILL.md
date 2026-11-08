---
name: taxonomically-informed-annotation-reranking
description: Use when MS/MS annotations from ISDB-DNP, Sirius or MS-Finder exist for
  spectra of a taxonomically known biological source and correct structures sit at
  low spectral-matching ranks, so re-ranking with taxonomic distance can boost rank-1
  recall.
when_to_use_negative:
- The biological source organism has no taxonomy resolvable against the Catalogue
  of Life (taxo bonuses cannot be computed for unresolved candidates)
- The analytes are ubiquitous/primary metabolites rather than specialized metabolites
  — the scoring 'will mostly benefit the annotation process of specialized metabolites
  and not ubiquitous molecules'
- You need a fair cross-tool performance comparison on compounds not uniformly present
  in the DNP — the benchmarking design favors ISDB-DNP since all benchmark compounds
  are in the DNP
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: taxo_scorer
  role: R scripts (1_taxo_cleaner.Rmd, 2_taxo_scorer.Rmd) performing cleaning, taxonomically
    informed scoring and re-ranking
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained TIMA pipeline implementing taxonomically informed metabolite annotation
    end-to-end
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: gnfinder
  role: Extracts taxonomic names from free-text biological-source fields and matches
    them against Global Names index / Catalogue of Life
  repo: https://github.com/gnames/gnfinder
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization used to find the optimal family/genus/species
    score combination
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
- name: ISDB-DNP
  role: Spectral matching tool backed by the Dictionary of Natural Products whose
    candidate list is re-ranked
  repo: http://oolonek.github.io/ISDB/
- name: Sirius
  role: Computational annotation tool whose output candidates are re-ranked by taxo-informed
    scoring
- name: MS-Finder
  role: Computational annotation tool whose output candidates are re-ranked by taxo-informed
    scoring
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:c9a38c629495e3882a3bfe17fcc2abc3bb8aa6b0f0f9f2ef74ff328d89a8bb87
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# taxonomically-informed-annotation-reranking

## Summary

Re-ranks computational metabolite annotation candidates (ISDB-DNP, Sirius, MS-Finder) by combining their normalized spectral scores with taxonomy-based bonuses at family, genus and species levels, exploiting the fact that specialized metabolites are strongly linked to the taxonomic position of the producing organism.

## When to use

Use when MS/MS annotations from ISDB-DNP, Sirius or MS-Finder exist for spectra of a taxonomically known biological source and correct structures sit at low spectral-matching ranks, so re-ranking with taxonomic distance can boost rank-1 recall.

## When NOT to use

- The biological source organism has no taxonomy resolvable against the Catalogue of Life (taxo bonuses cannot be computed for unresolved candidates)
- The analytes are ubiquitous/primary metabolites rather than specialized metabolites — the scoring 'will mostly benefit the annotation process of specialized metabolites and not ubiquitous molecules'
- You need a fair cross-tool performance comparison on compounds not uniformly present in the DNP — the benchmarking design favors ISDB-DNP since all benchmark compounds are in the DNP

## Inputs

- Annotation tool output tables (ISDB-DNP, Sirius, MS-Finder) with candidate structures and spectral scores
- MGF MS/MS spectra file (.mgf) for queried features
- Feature quantification table and sample metadata including organism identity
- Resolved taxonomic positions of organisms and candidate structures (e.g. against Catalogue of Life)
- Reference structure database with biological-source metadata (e.g. Dictionary of Natural Products v 27.1)

## Outputs

- Taxo-scored, re-ranked annotation table per feature (e.g. glaucium_taxo_annotations.csv)
- Rank-1 correct annotation counts per tool (e.g. taxo_scored_rank1_counts.csv)

## How to apply

First clean and standardize the annotation tool outputs and resolve candidate biological-source names against the Catalogue of Life backbone using gnfinder (1_taxo_cleaner.Rmd). Normalize each tool's original annotation score to the 0-1 range. Then apply the taxonomically informed scoring (2_taxo_scorer.Rmd) by adding the optimized weights at each resolved taxa level — e.g. 0.81 (family), 1.62 (genus), 2.55 (species) for ISDB-DNP — to the normalized spectral score, keeping the individual scores independent rather than hard-filtering candidates by taxonomy. Re-rank candidate structures per feature by the combined score and count correct annotations at rank 1; weights can be optimized with a parallelizable Bayesian optimization algorithm (ParBayesianOptimization). Verify success by comparing pre/post rank-1 counts (expected post-scoring: 1,510 ISDB-DNP, 1,508 Sirius, 546 MS-Finder on the 2,107-spectrum benchmark) and confirming F1-score gains.

## Related tools

- **taxo_scorer** (R scripts (1_taxo_cleaner.Rmd, 2_taxo_scorer.Rmd) performing cleaning, taxonomically informed scoring and re-ranking) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained TIMA pipeline implementing taxonomically informed metabolite annotation end-to-end) — https://github.com/taxonomicallyinformedannotation/tima
- **gnfinder** (Extracts taxonomic names from free-text biological-source fields and matches them against Global Names index / Catalogue of Life) — https://github.com/gnames/gnfinder
- **ParBayesianOptimization** (Parallelizable Bayesian optimization used to find the optimal family/genus/species score combination) — https://github.com/AnotherSamWilson/ParBayesianOptimization
- **ISDB-DNP** (Spectral matching tool backed by the Dictionary of Natural Products whose candidate list is re-ranked) — http://oolonek.github.io/ISDB/
- **Sirius** (Computational annotation tool whose output candidates are re-ranked by taxo-informed scoring)
- **MS-Finder** (Computational annotation tool whose output candidates are re-ranked by taxo-informed scoring)

## Examples

```
docker pull adafede/tima-r
```

## Evaluation signals

- After re-ranking, correct rank-1 counts on the 2,107-spectrum benchmarking dataset reach 1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder)
- F1 score of every evaluated tool increases systematically, with a reported 1.5- to 7-fold improvement
- Case-study check: for G. grandiflorum features m/z 342.1670 (RT 1.42 min) and m/z 356.1860 (RT 1.83 min), the correct structures (predicentrine, glaucine) move from spectral-matching ranks 9 and 7 to rank 1
- Every retained candidate has taxonomy resolved against the Catalogue of Life down to species level; unresolved entries were filtered out upstream
- Per-level weights land near the optimized values (0.81 family, 1.62 genus, 2.55 species for ISDB-DNP) when re-optimizing on the same training set

## Limitations

- The optimized score combination (0.81/1.62/2.55) is heavily dependent on the training set and 'should be taken with caution, and not as absolute optimal values'
- Taxonomically informed scoring mostly benefits specialized metabolites, not ubiquitous molecules
- Benchmarking does not allow fair tool comparison because all benchmark compounds are in the DNP (favoring ISDB-DNP), and GNPS spectral libraries were also part of the Sirius training set
- 197 benchmarking entries had large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; these problematic entries cannot be identified by the annotation tools (list in Problematic_entries.csv)

## Evidence

- [abstract] abstract: "the consideration of the taxonomic distance allowed an efficient re-ranking of the candidate structures leading to a systematic enhancement of the recall and precision rates"
- [discussion] optimal weights: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] workflow scripts: "1) cleaning and standardization of the outputs (1_taxo_cleaner.Rmd)"
- [discussion] gnfinder usage: "In order to extract taxonomic information out of the free text contents, the gnfinder program"
- [readme] tima purpose: "This repository contains everything needed to perform **T**axonomically **I**nformed **M**etabolite **A**nnotation."
