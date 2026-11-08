---
name: spectral-annotation-performance-analysis
description: Use when ranked candidate annotation lists from annotation tools must
  be scored against known ground-truth structures to quantify whether a re-ranking
  strategy improves precision, recall and F1.
when_to_use_negative:
- The goal is a basic taxonomic filtering process rather than a scoring/re-ranking
  system that keeps spectral similarity and taxonomic scores independent
- Benchmarking would be used to fairly compare tool performance when all compounds
  are in the DNP, since this favors the ISDB-DNP tool backed by the same database
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3360
tools:
- name: taxo_scorer
  role: Applies taxonomically informed scoring and re-ranking of candidate annotations
    before metric recomputation
  repo: https://github.com/oolonek/taxo_scorer
- name: ISDB-DNP
  role: Benchmarked annotation tool whose ranked candidate lists are scored pre/post
    re-ranking
- name: Sirius
  role: Benchmarked annotation tool whose ranked candidate lists are scored pre/post
    re-ranking
- name: MS-Finder
  role: Benchmarked annotation tool whose ranked candidate lists are scored pre/post
    re-ranking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:aa0042b5d9ff3f3f1c71c3a2b86461358e3fb9783e6bcdb8cf4c2863ca8a892a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - Sirius
  - taxo_scorer
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spectral-annotation-performance-analysis

## Summary

Benchmark the precision, recall and F1 score (correct-at-rank-1 as true positives) of computational metabolite annotation tools (ISDB-DNP, Sirius, MS-Finder) on a 2,107-spectrum benchmarking dataset, before and after taxonomically informed re-ranking.

## When to use

Use when ranked candidate annotation lists from annotation tools must be scored against known ground-truth structures to quantify whether a re-ranking strategy improves precision, recall and F1.

## When NOT to use

- The goal is a basic taxonomic filtering process rather than a scoring/re-ranking system that keeps spectral similarity and taxonomic scores independent
- Benchmarking would be used to fairly compare tool performance when all compounds are in the DNP, since this favors the ISDB-DNP tool backed by the same database

## Inputs

- Ranked candidate annotation lists from ISDB-DNP, Sirius and MS-Finder (.mgf-derived, 2,107 benchmarking entries)
- Ground-truth structure assignments for each benchmarking spectrum
- Biological source taxonomy per entry, resolved against the Catalogue of Life down to species level

## Outputs

- benchmark_metrics.csv with per-tool precision, recall, F1 before and after re-ranking and F1 fold-increase
- Rank-1 correct annotation counts (e.g. 1,510 ISDB-DNP, 1,508 Sirius, 546 MS-Finder after re-ranking)

## How to apply

Parse the ranked candidate lists produced by ISDB-DNP, Sirius and MS-Finder for the 2,107 benchmarking entries downloaded from the OSF deposit (https://osf.io/bvs6x/). For each tool, compute baseline TP (correct structure at rank 1), FP (wrong structure at rank 1) and FN (correct structure at rank >1), then derive precision, recall and F1. Apply taxonomically informed re-ranking with the optimized score combination 0.81 (family), 1.62 (genus), 2.55 (species) and recompute the same metrics on the re-ranked lists. Finally compute the F1 fold-increase (post/pre) per tool and compare against the reported 7-fold (ISDB-DNP), 1.5-fold (Sirius) and 3-fold (MS-Finder) increases, writing before/after metrics and fold-increases to benchmark_metrics.csv.

## Related tools

- **taxo_scorer** (Applies taxonomically informed scoring and re-ranking of candidate annotations before metric recomputation) — https://github.com/oolonek/taxo_scorer
- **ISDB-DNP** (Benchmarked annotation tool whose ranked candidate lists are scored pre/post re-ranking)
- **Sirius** (Benchmarked annotation tool whose ranked candidate lists are scored pre/post re-ranking)
- **MS-Finder** (Benchmarked annotation tool whose ranked candidate lists are scored pre/post re-ranking)

## Examples

```
docker pull adafede/tima-r
```

## Evaluation signals

- Rank-1 correct annotations after re-ranking match the reported values: 1,510 (ISDB-DNP), 1,508 (Sirius) and 546 (MS-Finder)
- Per-tool F1 fold-increase (post/pre) matches the reported 7-fold (ISDB-DNP), 1.5-fold (Sirius) and 3-fold (MS-Finder)
- Baseline TP/FP/FN partition is consistent: TP + FP = 2,107 entries and FN counts only correct structures ranked below rank 1
- benchmark_metrics.csv contains both before and after precision, recall, F1 and the computed fold-increase per tool
- Re-ranking uses the exact optimized score combination 0.81 (family), 1.62 (genus) and 2.55 (species)

## Limitations

- All compounds of the benchmarking dataset are present in the DNP, so the ISDB-DNP tool, which is backed by the same database, is favored in the comparison
- The GNPS spectral libraries were also part of the Sirius training set, confounding tool comparison
- 197 benchmarking entries have large mass differences (> 0.01 Da) between experimental parent ion mass and calculated exact mass and are listed in Problematic_entries.csv
- The optimized score combination is heavily dependent on the training sets and should not be taken as absolute optimal values
- Taxonomically informed scoring mostly benefits the annotation of specialized metabolites, not ubiquitous molecules

## Evidence

- [discussion] F1 fold increase on benchmarking dataset: "was shown to systematically improve the F1 score of the evaluated solutions (ISDB-DNP, Sirius, MS-Finder) with a 1.5 to 7-fold increase"
- [discussion] Benchmark dataset size: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] Optimized score combination: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] Data availability for reproduction: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
- [readme] Scoring script purpose: [evidence span withheld — anchor status: external_doc]
