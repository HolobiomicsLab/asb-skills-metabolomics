---
name: metabolite-annotation-benchmarking
description: Use when ranked candidate annotation lists exist for MS/MS spectra and
  you need to quantify annotation quality (precision/recall/F1 with correct-at-rank-1
  as TP) or measure the impact of a re-ranking strategy such as taxonomically informed
  scoring.
when_to_use_negative:
- The analyzed molecules are ubiquitous rather than specialized metabolites — taxonomically
  informed scoring mostly benefits specialized metabolites
- You need a fair head-to-head comparison of tool performance — all benchmark compounds
  are present in the DNP, which by definition favors ISDB-DNP, and GNPS spectral libraries
  were part of the Sirius training set
- Entries have no biological source taxonomy resolvable against the Catalogue of Life
  down to species level, so taxonomic distance cannot be computed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: taxo_scorer
  role: Performs the taxonomically informed scoring and re-ranking of candidate annotations
    (2_taxo_scorer.Rmd) whose effect on precision/recall/F1 is benchmarked
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained repository containing everything needed to perform Taxonomically
    Informed Metabolite Annotation, including the benchmarking scripts
  repo: https://github.com/taxonomicallyinformedannotation/tima
- name: ISDB-DNP
  role: Evaluated annotation tool backed by the Dictionary of Natural Products (DNP
    v 27.1); showed the largest F1 fold-increase (7-fold)
- name: Sirius
  role: Evaluated fragmentation-tree-based annotation tool; showed a 1.5-fold F1 increase
    after taxonomically informed re-ranking
- name: MS-Finder
  role: Evaluated combined-parameter annotation tool; showed a 3-fold F1 increase
    after taxonomically informed re-ranking
- name: gnfinder
  role: Extracts taxonomic information from free text and matches detected names against
    the Global Names index / Catalogue of Life, feeding the taxonomic dimension used
    in re-ranking
  repo: https://github.com/gnames/gnfinder
- name: ParBayesianOptimization
  role: Parallelizable Bayesian optimization used to find the optimal taxonomic score
    combination (0.81, 1.62, 2.55 for family, genus, species)
  repo: https://github.com/AnotherSamWilson/ParBayesianOptimization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:0eb6539145ce9cc4f2ae9a31138a8bfb681c9209eef2be1128d7aa6e08075648
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metabolite-annotation-benchmarking

## Summary

Benchmark computational metabolite annotation tools (ISDB-DNP, Sirius, MS-Finder) by computing precision, recall and F1 (rank-1 correctness as true positives) on the 2,107-spectrum benchmarking dataset, before and after taxonomically informed re-ranking, to quantify F1 fold-increase per tool.

## When to use

Use when ranked candidate annotation lists exist for MS/MS spectra and you need to quantify annotation quality (precision/recall/F1 with correct-at-rank-1 as TP) or measure the impact of a re-ranking strategy such as taxonomically informed scoring.

## When NOT to use

- The analyzed molecules are ubiquitous rather than specialized metabolites — taxonomically informed scoring mostly benefits specialized metabolites
- You need a fair head-to-head comparison of tool performance — all benchmark compounds are present in the DNP, which by definition favors ISDB-DNP, and GNPS spectral libraries were part of the Sirius training set
- Entries have no biological source taxonomy resolvable against the Catalogue of Life down to species level, so taxonomic distance cannot be computed

## Inputs

- Ranked candidate annotation lists (outputs) from ISDB-DNP, Sirius and MS-Finder for the 2,107 benchmarking entries (from OSF deposit osf.io/bvs6x/)
- Benchmarking dataset input data (2,107 distinct entries) from the OSF deposit
- Taxonomic information (species, genus, family) for the biological source of each entry, resolved against the Catalogue of Life

## Outputs

- benchmark_metrics.csv with before/after precision, recall, F1 and F1 fold-increase per tool
- Per-tool TP, FP, FN counts at baseline and after taxonomically informed re-ranking
- Per-tool rank-1 correct annotation counts (reported: 1,510 ISDB-DNP, 1,508 Sirius, 546 MS-Finder)

## How to apply

Download the benchmarking dataset (input and output data) from the OSF deposit https://osf.io/bvs6x/ and parse the ranked candidate annotation lists produced by ISDB-DNP, Sirius and MS-Finder for the 2,107 benchmarking entries. For each tool, define TP as a correct structure at rank 1, FP as a wrong structure at rank 1, and FN as the correct structure present at rank >1, then derive precision, recall and F1. Re-rank candidates with the taxo_scorer approach using the optimized taxonomically informed score combination of 0.81 (family), 1.62 (genus) and 2.55 (species) as established for ISDB-DNP, then recompute TP/FP/FN, precision, recall and F1 on the re-ranked lists. Report the F1 fold-increase (post/pre) per tool and compare against the reported values of 7-fold for ISDB-DNP, 1.5-fold for Sirius and 3-fold for MS-Finder, with rank-1 correct annotations rising to 1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder). Treat the optimized score weights as training-set dependent rather than absolute optima.

## Related tools

- **taxo_scorer** (Performs the taxonomically informed scoring and re-ranking of candidate annotations (2_taxo_scorer.Rmd) whose effect on precision/recall/F1 is benchmarked) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained repository containing everything needed to perform Taxonomically Informed Metabolite Annotation, including the benchmarking scripts) — https://github.com/taxonomicallyinformedannotation/tima
- **ISDB-DNP** (Evaluated annotation tool backed by the Dictionary of Natural Products (DNP v 27.1); showed the largest F1 fold-increase (7-fold))
- **Sirius** (Evaluated fragmentation-tree-based annotation tool; showed a 1.5-fold F1 increase after taxonomically informed re-ranking)
- **MS-Finder** (Evaluated combined-parameter annotation tool; showed a 3-fold F1 increase after taxonomically informed re-ranking)
- **gnfinder** (Extracts taxonomic information from free text and matches detected names against the Global Names index / Catalogue of Life, feeding the taxonomic dimension used in re-ranking) — https://github.com/gnames/gnfinder
- **ParBayesianOptimization** (Parallelizable Bayesian optimization used to find the optimal taxonomic score combination (0.81, 1.62, 2.55 for family, genus, species)) — https://github.com/AnotherSamWilson/ParBayesianOptimization

## Examples

```
docker pull adafede/tima-r
```

## Evaluation signals

- Baseline rank-1 correct counts reproduce the paper's numbers after re-ranking: 1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder) out of 2,107 entries
- Computed F1 fold-increase per tool matches the reported 7-fold (ISDB-DNP), 1.5-fold (Sirius) and 3-fold (MS-Finder)
- TP/FP/FN partition is exhaustive per entry: exactly one of correct-at-rank-1 (TP), wrong-at-rank-1 (FP), or correct-at-rank->1 (FN)
- Re-ranking uses the optimized taxonomic score combination 0.81 (family), 1.62 (genus), 2.55 (species); changing weights away from these values degrades the fold-increase
- All metrics are computed on exactly 2,107 distinct benchmarking entries; entry count mismatch flags an incorrect parse of the OSF inputs

## Limitations

- The optimized score combination (0.81, 1.62, 2.55) is heavily dependent on the training sets and should not be taken as absolute optimal values
- All compounds of the benchmarking dataset are present in the DNP, so the benchmark does not allow a fair comparison of tool performances and favors ISDB-DNP
- GNPS spectral libraries were also part of the Sirius training set, confounding tool comparison
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the list is provided as Problematic_entries.csv but erroneous entries cannot be identified by annotation tools
- Taxonomically informed scoring will mostly benefit the annotation process of specialized metabolites and not ubiquitous molecules

## Evidence

- [abstract] F1 increase magnitudes: "the consideration of the taxonomic distance allowed an efficient re-ranking of the candidate structures leading to a systematic enhancement of the recall and precision rates of the tools (1.5- to"
- [discussion] benchmarking dataset size: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] optimized score weights: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] re-ranking script: "taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
- [discussion] data availability: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
- [other] scripts location: "Scripts are available at https://github.com/oolonek/taxo_scorer"
