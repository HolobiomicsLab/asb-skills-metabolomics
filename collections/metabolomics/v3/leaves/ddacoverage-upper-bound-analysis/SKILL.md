---
name: ddacoverage-upper-bound-analysis
description: Use when you need to know how many ions an ideal DDA controller could
  fragment at least once in a given injection — e.g. before designing or evaluating
  a new DDA method, to quantify headroom above observed TopN coverage.
when_to_use_negative:
- When you already have a final feature/intensity matrix and only need standard statistical
  analysis — this skill is specifically about acquisition-level upper bounds, not
  downstream data analysis.
- When the acquisition is DIA/SWATH rather than DDA, since the bipartite MS2-scan-slot
  model assumes a data-dependent scan schedule.
- When you lack access to the MS1 survey scan schedule or scan times, as the edge-condition
  (MS2 scan time inside bounding box plus preceding MS1 scan RT) cannot be evaluated
  without them.
edam_operation: http://edamontology.org/operation_2949
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: In silico framework used to prototype, optimize and validate the DDA strategies
    whose coverage upper bound this analysis quantifies; it supplies the virtual MS
    scan schedule and can simulate alternative controllers against the computed bound.
- name: IAPI
  role: Instrument Application Programming Interface used to apply the developed DDA
    methods on real samples, whose coverage can be benchmarked against the maximum-matching
    upper bound computed here.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:ca0ea533c13ed2be8ca68197fd07a3bba927cd4ae89707e4fb51960d5dcd079c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - ViMMS
  edam_operation: http://edamontology.org/operation_2949
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ddacoverage-upper-bound-analysis

## Summary

Computes the theoretically optimal DDA coverage for a given LC-MS/MS injection by casting peak-fragmentation as a bipartite maximum-matching problem between MS2 scan slots and picked-peak bounding boxes, establishing the upper bound against which any DDA strategy (e.g. TopN) can be compared.

## When to use

Use when you need to know how many ions an ideal DDA controller could fragment at least once in a given injection — e.g. before designing or evaluating a new DDA method, to quantify headroom above observed TopN coverage.

## When NOT to use

- When you already have a final feature/intensity matrix and only need standard statistical analysis — this skill is specifically about acquisition-level upper bounds, not downstream data analysis.
- When the acquisition is DIA/SWATH rather than DDA, since the bipartite MS2-scan-slot model assumes a data-dependent scan schedule.
- When you lack access to the MS1 survey scan schedule or scan times, as the edge-condition (MS2 scan time inside bounding box plus preceding MS1 scan RT) cannot be evaluated without them.

## Inputs

- Full-scan mzML file(s) for the sample of interest (for peak picking)
- TopN DDA mzML file from which mean MS1 and MS2 scan times and the fixed N (scans per survey scan) are extracted
- Peak bounding-box list (min/max RT and m/z per picked peak) exported from peak picking (e.g. MZmine 2)
- Minimum MS1 intensity threshold required for a peak to be eligible for fragmentation

## Outputs

- Optimal DDA coverage upper bound (maximum matching size, i.e. number of peaks fragmentable at least once)
- Bipartite graph of MS2 scans vs peak bounding boxes with eligibility edges
- Comparison of optimal vs observed TopN coverage (e.g. serum: 1542 optimal vs 656 observed; beer: 2955 optimal vs 1046 observed)

## How to apply

First pick peaks from the full-scan mzML files (e.g. using MZmine 2 with the parameters in Supporting Information Table SI-1) and export each peak as a bounding box of min/max retention time and m/z. Next, extract the mean MS1 and MS2 scan times from the TopN mzML file together with the fixed N of MS2 scans per MS1 survey scan to reconstruct the MS scan schedule and each scan's start time. Build a bipartite graph whose two node sets are MS2 scans and peak bounding boxes; add an edge only when the MS2 scan time falls inside the bounding box's RT limits, the preceding MS1 scan RT is also inside the box, and the peak intensity in that MS1 scan exceeds the minimum MS1 intensity required for fragmentation. Run Hopcroft–Karp maximum matching (e.g. via scipy.sparse.csgraph.maximum_bipartite_matching) on this graph — the matching size is the optimal coverage upper bound. For the paper's serum and beer extracts this yields 1542 and 2955 fragmentable peaks respectively, versus TopN's observed coverage of only 656 and 1046.

## Related tools

- **ViMMS** (In silico framework used to prototype, optimize and validate the DDA strategies whose coverage upper bound this analysis quantifies; it supplies the virtual MS scan schedule and can simulate alternative controllers against the computed bound.)
- **IAPI** (Instrument Application Programming Interface used to apply the developed DDA methods on real samples, whose coverage can be benchmarked against the maximum-matching upper bound computed here.)

## Examples

```
from scipy.sparse.csgraph import maximum_bipartite_matching; match = maximum_bipartite_matching(graph, perm_type='column'); n_optimal = (match != -1).sum()
```

## Evaluation signals

- The computed maximum-matching size must reproduce the reported optima: 1542 peaks for serum and 2955 for beer.
- The observed TopN coverage (656 serum, 1046 beer) must be strictly less than the computed upper bound for the same injection.
- Edges in the bipartite graph must satisfy all three conditions simultaneously: MS2 scan time within bounding-box RT limits, preceding MS1 scan RT within the box, and MS1 peak intensity above the minimum fragmentation threshold.
- The scan schedule derived from mean MS1/MS2 times and fixed N must align with the actual TopN mzML scan ordering.
- Peak bounding boxes exported from the picker must be non-empty and cover the expected RT/m/z ranges of the picked peaks.

## Limitations

- The upper bound assumes a fixed scan schedule (mean MS1 and MS2 times, fixed N) and does not model adaptive scheduling changes a real controller could make.
- Edge construction depends on a minimum MS1 intensity threshold for fragmentation eligibility; different thresholds change the bound.
- The analysis is per-injection and assumes ions elute unevenly; it does not extrapolate to coverage across multiple injections.

## Evidence

- [abstract] theoretical upper bound on DDA coverage via bipartite matching: "we first provide theoretical results that show how much improvement is possible over current DDA strategies"
- [methods] peak picking and bounding boxes from full-scan mzML: [evidence span withheld — anchor status: not_found]
- [methods] bipartite graph edge conditions: [evidence span withheld — anchor status: not_found]
- [methods] Hopcroft–Karp algorithm and expected counts: [evidence span withheld — anchor status: not_found]
- [results] observed vs optimal coverage comparison: [evidence span withheld — anchor status: not_found]
- [intro] objective framing from intro: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
