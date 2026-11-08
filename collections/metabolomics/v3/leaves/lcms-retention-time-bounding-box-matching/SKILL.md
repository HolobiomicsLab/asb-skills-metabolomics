---
name: lcms-retention-time-bounding-box-matching
description: Use when you have full-scan mzML peak-picked bounding boxes (min/max
  RT and m/z) plus a TopN mzML schedule and need the upper bound on fragmentable peaks
  for a single injection.
when_to_use_negative:
- You already have real DDA coverage measurements and only want to characterize observed
  fragmentation, not the theoretical upper bound
- The input is a feature/peak-intensity table without bounding-box RT and m/z limits,
  so scan-in-box eligibility edges cannot be formed
- The aim is to design or benchmark new DDA acquisition strategies in silico with
  ViMMS rather than to compute a per-injection coverage upper bound
edam_operation: http://edamontology.org/operation_3949
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to simulate the injections and
    generate the mzML data on which the matching upper bound is evaluated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:0eb603e7ce7d68b0879c372b564151dcbe4a79bb6238ac249215a849deb3dea4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ViMMS
  edam_operation: http://edamontology.org/operation_3949
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lcms-retention-time-bounding-box-matching

## Summary

Computes the theoretically optimal DDA coverage for a given injection by building a bipartite graph between MS2 scan slots and picked-peak bounding boxes and running Hopcroft–Karp maximum matching, establishing an upper bound on the number of ions any DDA method could fragment at least once.

## When to use

Use when you have full-scan mzML peak-picked bounding boxes (min/max RT and m/z) plus a TopN mzML schedule and need the upper bound on fragmentable peaks for a single injection.

## When NOT to use

- You already have real DDA coverage measurements and only want to characterize observed fragmentation, not the theoretical upper bound
- The input is a feature/peak-intensity table without bounding-box RT and m/z limits, so scan-in-box eligibility edges cannot be formed
- The aim is to design or benchmark new DDA acquisition strategies in silico with ViMMS rather than to compute a per-injection coverage upper bound

## Inputs

- Full-scan (MS1) mzML files for the sample extracts (e.g., serum and beer)
- TopN DDA mzML file from the same injection providing MS1/MS2 scan times
- Peak bounding boxes (min/max retention time and m/z) exported from MZmine 2 per SI Table SI-1 parameters
- Fixed TopN value of N (number of MS2 scans per MS1 survey scan)
- Minimum MS1 intensity threshold for fragmentation eligibility

## Outputs

- Maximum-matching count of fragmentable peak bounding boxes per sample (optimal coverage)
- Observed TopN coverage counts per sample for comparison
- optimal_coverage_comparison.csv with optimal-versus-observed coverage values for both extracts

## How to apply

Pick peaks from the full-scan mzML files (e.g., serum and beer extracts) using MZmine 2 with parameters per Supporting Information Table SI-1 and export each peak as a bounding box of min/max retention time and m/z. Extract mean MS1 and MS2 scan times from the TopN mzML file together with the fixed value of N (MS2 scans per MS1 survey scan) to reconstruct the scan schedule with respective scan start times. Build a bipartite graph with one node set of MS2 scans and one of peak bounding boxes, adding an edge only when the MS2 scan time falls within the bounding-box RT limits, the preceding MS1 scan RT is also within the box, and the peak intensity in that MS1 scan exceeds the minimum MS1 intensity for fragmentation. Run the Hopcroft–Karp maximum-matching algorithm (e.g., via scipy.sparse.csgraph) to obtain the maximum number of fragmentable peak-bounding boxes, then compare against observed TopN coverage (expected: 1542 optimal vs 656 observed for serum; 2955 optimal vs 1046 observed for beer).

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to simulate the injections and generate the mzML data on which the matching upper bound is evaluated)

## Examples

```
from scipy.sparse.csgraph import maximum_bipartite_matching; match = maximum_bipartite_matching(bipartite_graph, perm_type='column')
```

## Evaluation signals

- Maximum-matching count must equal 1542 peaks for serum and 2955 peaks for beer
- Reproduces the reported gap: 656 observed vs 1542 optimal (serum) and 1046 observed vs 1046/2955 optimal (beer)
- Every matched edge satisfies all three eligibility conditions: MS2 scan time within bounding-box RT limits, preceding MS1 RT within the box, and MS1 peak intensity above the minimum fragmentation threshold
- Matching size is monotonically non-decreasing when eligibility constraints are relaxed (sanity check on graph construction)

## Limitations

- The result is an upper bound for a particular injection, not an achievable method; it assumes perfect scheduling decisions and relies on the reconstructed scan schedule being faithful to the real instrument timing
- Coverage depends on MZmine 2 peak-picking parameters (SI Table SI-1) and the chosen minimum MS1 intensity for fragmentation; different parameters change the bounding boxes and hence the bound

## Evidence

- [methods] Using a bipartite graph maximum matching (Hopcroft–Karp) between MS2 scan slots and picked-peak bounding boxes: [evidence span withheld — anchor status: not_found]
- [results] optimal coverage upper bound is 1542 peaks for serum and 2955 for beer: [evidence span withheld — anchor status: not_found]
- [intro] We start by computing the theoretically optimal performance for any particular injection: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
