---
name: precision-recall-f1-computation
description: Use when you have ranked candidate annotation lists per spectrum (e.g.
  ISDB-DNP, Sirius, MS-Finder outputs) and ground-truth structures, and need to compare
  tool performance or quantify the effect of re-ranking.
when_to_use_negative:
- When candidate lists are not ranked or the correct structure is absent from all
  candidates — FN/TP definitions at rank 1 are then ill-defined
- 'When the goal is a fair cross-tool comparison: all benchmark compounds are present
  in the DNP, so ISDB-DNP is favored by definition and GNPS libraries were part of
  the Sirius training set'
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: taxo_scorer
  role: Re-ranks candidate annotations using taxonomic distance so that before/after
    precision, recall and F1 can be computed and compared
  repo: https://github.com/oolonek/taxo_scorer
- name: ISDB-DNP
  role: Annotation tool whose ranked candidate lists are evaluated for TP/FP/FN and
    F1 fold-increase
  repo: http://oolonek.github.io/ISDB/
- name: Sirius
  role: Annotation tool whose ranked candidate lists are evaluated for TP/FP/FN and
    F1 fold-increase
- name: MS-Finder
  role: Annotation tool whose ranked candidate lists are evaluated for TP/FP/FN and
    F1 fold-increase
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:bfa4abe5c64f03eda24431ae168512d8f91c45ec527beeec65c32fabbc6d5f74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oolonek/taxo_scorer
  tools:
  - ISDB-DNP
  - MS-Finder
  - Sirius
  - taxo_scorer
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# precision-recall-f1-computation

## Summary

Computes precision, recall and F1 score for metabolite annotation tools by treating a correct structure at rank 1 as a true positive, before and after re-ranking (e.g. taxonomically informed scoring), to quantify annotation quality on a benchmark set.

## When to use

Use when you have ranked candidate annotation lists per spectrum (e.g. ISDB-DNP, Sirius, MS-Finder outputs) and ground-truth structures, and need to compare tool performance or quantify the effect of re-ranking.

## When NOT to use

- When candidate lists are not ranked or the correct structure is absent from all candidates — FN/TP definitions at rank 1 are then ill-defined
- When the goal is a fair cross-tool comparison: all benchmark compounds are present in the DNP, so ISDB-DNP is favored by definition and GNPS libraries were part of the Sirius training set

## Inputs

- Ranked candidate annotation lists per spectrum from ISDB-DNP, Sirius and MS-Finder (OSF deposit osf.io/bvs6x)
- Ground-truth reference structures for the 2,107-spectrum benchmarking dataset
- Taxonomically informed re-ranked candidate lists (post-scoring, e.g. via 2_taxo_scorer.Rmd with 0.81/1.62/2.55 weights)

## Outputs

- Per-tool TP, FP, FN counts at rank 1 (baseline and re-ranked)
- Per-tool precision, recall and F1 score (baseline and re-ranked)
- Per-tool F1 fold-increase (post/pre), e.g. benchmark_metrics.csv

## How to apply

Parse the ranked candidate lists for each of the 2,107 benchmarking entries downloaded from the OSF deposit (osf.io/bvs6x). For each entry and tool, define TP as the correct structure at rank 1, FP as a wrong structure at rank 1, and FN as the correct structure present but at rank >1; derive precision = TP/(TP+FP), recall = TP/(TP+FN) and F1 = 2·precision·recall/(precision+recall). To measure the effect of re-ranking, apply taxonomically informed scoring (optimized weights 0.81 family, 1.62 genus, 2.55 species for ISDB-DNP) to re-order candidates, then recompute TP/FP/FN and all three metrics identically on the re-ranked lists. Finally compute the per-tool F1 fold-increase (post/pre) and report before/after metrics, e.g. a 7-fold increase for ISDB-DNP, 1.5-fold for Sirius and 3-fold for MS-Finder, with rank-1 correct annotations rising to 1,510, 1,508 and 546 respectively.

## Related tools

- **taxo_scorer** (Re-ranks candidate annotations using taxonomic distance so that before/after precision, recall and F1 can be computed and compared) — https://github.com/oolonek/taxo_scorer
- **ISDB-DNP** (Annotation tool whose ranked candidate lists are evaluated for TP/FP/FN and F1 fold-increase) — http://oolonek.github.io/ISDB/
- **Sirius** (Annotation tool whose ranked candidate lists are evaluated for TP/FP/FN and F1 fold-increase)
- **MS-Finder** (Annotation tool whose ranked candidate lists are evaluated for TP/FP/FN and F1 fold-increase)

## Examples

```
Rscript 2_taxo_scorer.Rmd  # then compute TP/FP/FN at rank 1, precision, recall, F1 per tool and write benchmark_metrics.csv
```

## Evaluation signals

- TP+FN must equal the number of benchmark entries (2,107) with a correct structure present among candidates for each tool
- Baseline rank-1 correct counts should reproduce the paper's reported post-re-ranking values: 1,510 (ISDB-DNP), 1,508 (Sirius), 546 (MS-Finder)
- F1 fold-increase (post/pre) should match approximately 7-fold for ISDB-DNP, 1.5-fold for Sirius and 3-fold for MS-Finder
- Precision, recall and F1 must all lie in [0,1] and be non-decreasing after re-ranking for all three tools (systematic increase reported)
- All before/after metrics and fold-increases are written to a single benchmark_metrics.csv without missing per-entry values

## Limitations

- The 197 benchmarking entries with large mass differences (>0.01 Da) between experimental parent mass and calculated exact mass cannot be identified as erroneous by the annotation tools, biasing TP/FP counts
- A correct-at-rank-1 definition ignores partially correct lower-ranked candidates; only the top rank counts as a true positive
- The comparison is not fair across tools: ISDB-DNP is backed by the DNP and the GNPS spectral libraries were part of the Sirius training set
- F1 improvements mostly benefit specialized metabolites, not ubiquitous molecules

## Evidence

- [discussion] Task card finding on rank-1 TP definition: [evidence span withheld — anchor status: not_found]
- [discussion] Optimized score weights for re-ranking: "the optimal combination of scores was 0.81, 1.62 and 2.55 (family, genus and species taxa level, respectively)"
- [discussion] Benchmark dataset size and tool scope: "benchmark the impact of taxonomic distance consideration on a set of 2,107 identified molecules using three different computational mass"
- [discussion] Data availability for reproducing metrics: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
- [discussion] Re-ranking workflow step: "2) taxonomically informed scoring and re-ranking (2_taxo_scorer.Rmd)"
