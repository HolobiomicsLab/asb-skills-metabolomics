---
name: bipartite-graph-construction-from-scan-schedule
description: Use when you have full-scan mzML files with picked peak bounding boxes
  and a TopN mzML with a known N (MS2 scans per MS1) and want the theoretical optimum
  of fragmentable ions for that injection.
when_to_use_negative:
- The goal is to measure actual observed DDA coverage from a real acquisition rather
  than the theoretical upper bound for any possible method
- No peak bounding boxes can be exported (e.g., only fragmented MS2 spectra without
  full-scan peak picking with MZmine 2), so RT/mz eligibility edges cannot be defined
- The acquisition is not TopN-style with a definable MS2-per-MS1 scan schedule (e.g.,
  DIA or scheduled inclusion-list methods)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MZmine 2
  role: Peak-picks the full-scan mzML files and exports each peak as a bounding box
    of min/max retention time and m/z
- name: scipy.sparse.csgraph
  role: Provides the Hopcroft–Karp maximum-matching implementation used to solve the
    bipartite graph
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and validate the
    DDA acquisition strategies whose scan schedules this graph models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:e46e93e58f69edbf889a4a9419ebfe8277434fbeb58e345c1c305140d3f6abb4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  - scipy.sparse.csgraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bipartite-graph-construction-from-scan-schedule

## Summary

Constructs a bipartite graph between MS2 scan slots (from a TopN scan schedule) and picked-peak bounding boxes (RT/mz min-max boxes from MZmine 2) and solves maximum matching (Hopcroft–Karp) to compute the theoretical upper bound on the number of ions any DDA method could fragment in a single injection.

## When to use

Use when you have full-scan mzML files with picked peak bounding boxes and a TopN mzML with a known N (MS2 scans per MS1) and want the theoretical optimum of fragmentable ions for that injection.

## When NOT to use

- The goal is to measure actual observed DDA coverage from a real acquisition rather than the theoretical upper bound for any possible method
- No peak bounding boxes can be exported (e.g., only fragmented MS2 spectra without full-scan peak picking with MZmine 2), so RT/mz eligibility edges cannot be defined
- The acquisition is not TopN-style with a definable MS2-per-MS1 scan schedule (e.g., DIA or scheduled inclusion-list methods)

## Inputs

- Full-scan MS1 mzML files (serum and beer extracts) for peak picking
- TopN DDA mzML file from which mean MS1/MS2 scan times are extracted
- Fixed value of N (number of MS2 scans per MS1 survey scan)
- MZmine 2 peak-picking parameters (Supporting Information Table SI-1)
- Minimum MS1 intensity threshold required for fragmentation

## Outputs

- Bipartite graph of MS2 scan slots vs peak bounding boxes with eligibility edges
- Maximum-matching count of fragmentable peak boxes per sample (1542 serum, 2955 beer)
- optimal_coverage_comparison.csv with optimal vs observed TopN coverage (656 serum, 1046 beer)

## How to apply

Pick peaks from the full-scan mzML files with MZmine 2 (parameters per Supporting Information Table SI-1) and export each peak as a bounding box of min/max retention time and m/z. Build the MS scan schedule by extracting mean MS1 and MS2 scan times from the TopN mzML and the fixed value of N, giving each MS2 slot its scan start time. Add an edge between an MS2 scan and a peak box only when the MS2 scan time falls within the box's RT limits, the preceding MS1 scan RT is also within the box, and the peak intensity in that MS1 scan exceeds the minimum MS1 intensity for fragmentation. Run Hopcroft–Karp maximum matching (e.g., via scipy.sparse.csgraph) to get the maximum number of fragmentable peak boxes, then compare with observed TopN coverage (serum: 656 observed vs 1542 optimal; beer: 1046 observed vs 2955 optimal).

## Related tools

- **MZmine 2** (Peak-picks the full-scan mzML files and exports each peak as a bounding box of min/max retention time and m/z)
- **scipy.sparse.csgraph** (Provides the Hopcroft–Karp maximum-matching implementation used to solve the bipartite graph)
- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and validate the DDA acquisition strategies whose scan schedules this graph models)

## Examples

```
from scipy.sparse.csgraph import maximum_bipartite_matching; match = maximum_bipartite_matching(biadjacency, perm_type='column'); print(match.max() + 1)
```

## Evaluation signals

- Maximum-matching counts must equal 1542 peaks for serum and 2955 peaks for beer
- The reproduced gap must be 656 observed vs 1542 optimal (serum) and 1046 observed vs 2952 optimal (beer), i.e., matching observed TopN coverage of 656 and 1046
- Every edge in the graph must satisfy all three eligibility conditions: MS2 scan time within the bounding-box RT limits, preceding MS1 scan RT within the box, and MS1 intensity above the fragmentation threshold
- The matching size must never exceed the number of peak bounding boxes or the number of MS2 scan slots
- optimal_coverage_comparison.csv must contain one optimal-vs-observed pair per extract

## Limitations

- The result is an upper bound on coverage for a particular injection, not an achievable method performance; real controllers cannot always attain the matching
- Edge eligibility depends on MZmine 2 peak-picking parameters (SI Table SI-1) and the minimum MS1 intensity for fragmentation, so different settings change the optimum
- The scan schedule assumes a fixed N (MS2 scans per MS1 survey scan) and mean scan times, which may not reflect dynamic real acquisitions

## Evidence

- [abstract] theoretical upper bound: "we first provide theoretical results that show how much improvement is possible over current DDA strategies"
- [supplementary] bipartite construction: [evidence span withheld — anchor status: not_found]
- [supplementary] edge eligibility: [evidence span withheld — anchor status: not_found]
- [supplementary] peak picking input: [evidence span withheld — anchor status: not_found]
- [supplementary] expected results: [evidence span withheld — anchor status: not_found]
