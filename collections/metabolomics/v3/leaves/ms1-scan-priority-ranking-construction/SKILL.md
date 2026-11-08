---
name: ms1-scan-priority-ranking-construction
description: Use when an mzML full-scan dataset plus a prior TopN MS1/MS2 scan schedule
  must be turned into a per-scan ranked MS2 fragmentation priority list via weighted
  (nonbinary) ion intensities.
when_to_use_negative:
- The goal is binary dynamic exclusion (fragment/exclude only) rather than continuous
  prioritization — the WeightedDEW weight collapses to the standard DEW case when
  only the t0 exclusion window matters
- Input is already a processed feature table with no per-MS1-scan timing or prior
  MS2 fragmentation schedule, so tf per m/z cannot be determined
- The acquisition is DIA/SWATH, where ion prioritization decisions per scan do not
  apply
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype, optimize, and simulate
    the WeightedDEW priority-ranking strategy in silico before validation on real
    hardware
- name: MZmine 2
  role: Peak detection/processing of full-scan mzML data to obtain ion bounding boxes
    and intensities feeding the ranking
- name: XCMS
  role: Alternative peak-picking tool for extracting MS1 ion intensities per scan
    from the mzML survey scans
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:39dc5ded73267d399b711613f8b3405e285cfedebd880e8346acc4802e0d7805
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ms1-scan-priority-ranking-construction

## Summary

Constructs a per-MS1-scan TopN fragmentation priority list by weighting each observed ion's survey-scan intensity with the WeightedDEW nonbinary dynamic-exclusion weight (0 for t0 seconds after fragmentation, then linear rise to 1 at t1), replacing binary include/exclude DDA decisions with a continuous prioritization ranking.

## When to use

Use when an mzML full-scan dataset plus a prior TopN MS1/MS2 scan schedule must be turned into a per-scan ranked MS2 fragmentation priority list via weighted (nonbinary) ion intensities.

## When NOT to use

- The goal is binary dynamic exclusion (fragment/exclude only) rather than continuous prioritization — the WeightedDEW weight collapses to the standard DEW case when only the t0 exclusion window matters
- Input is already a processed feature table with no per-MS1-scan timing or prior MS2 fragmentation schedule, so tf per m/z cannot be determined
- The acquisition is DIA/SWATH, where ion prioritization decisions per scan do not apply

## Inputs

- Full-scan mzML file with MS1 peak bounding boxes (e.g., serum QCA or beer QCB extract)
- TopN-derived MS1/MS2 scan schedule specifying prior fragmentation times tf per m/z
- WeightedDEW parameters: exclusion onset t0 (15 s), full-eligibility time t1 (120 s)
- Minimum MS1 intensity threshold for candidate ions
- Number of ions to fragment per MS1 scan (TopN, N)

## Outputs

- Per-MS1-scan priority ranking table with a weight column w in [0, 1] and weighted-intensity column for every ranked ion
- WeightedDEW MS2 fragmentation priority list (top N ions per MS1 scan)

## How to apply

Load the full-scan mzML peak bounding boxes and the TopN-derived MS1/MS2 scan schedule (e.g., the serum QCA and beer QCB extracts) — peak detection can be done with MZmine 2 or XCMS, and candidate strategies can be simulated in silico with ViMMS before touching hardware. For each MS1 scan and each ion above the minimum MS1 intensity threshold, determine tf, the most recent fragmentation time of that m/z, from the prior MS2 schedule. Compute the weight with the generalized dynamic-exclusion parameters t0 = 15 s and t1 = 120 s: w = 0 while t - tf <= t0, then a linear increase from 0 at t0 to 1 at t1 — this generalizes the standard dynamic exclusion window, which is recovered as the special case of full exclusion for t0 seconds. Multiply each ion's survey-scan intensity by its weight and rank ions in descending weighted-intensity order. Select the top N ions per MS1 scan as the WeightedDEW MS2 fragmentation priority list and emit the per-scan ranking table. The rationale is that continuous weighting lets partially excluded ions compete for fragmentation slots, improving DDA coverage for a single injection versus standard DDA.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype, optimize, and simulate the WeightedDEW priority-ranking strategy in silico before validation on real hardware)
- **MZmine 2** (Peak detection/processing of full-scan mzML data to obtain ion bounding boxes and intensities feeding the ranking)
- **XCMS** (Alternative peak-picking tool for extracting MS1 ion intensities per scan from the mzML survey scans)

## Examples

```
w = 0.0 if (t - tf) <= 15 else min((t - tf - 15) / (120 - 15), 1.0); rank ions by intensity * w descending per MS1 scan
```

## Evaluation signals

- Every ranked ion in the output carries a weight column with values in [0, 1]
- t0 = 15 s and t1 = 120 s are applied consistently: w = 0 for all t - tf <= 15 s
- Weight rises linearly from 0 at t0 to 1 at t1, matching the linear-rise behavior in the paper's Figure 3
- Exactly the top N ions by descending weighted intensity (survey intensity × w) are selected per MS1 scan
- tf for each ion corresponds to the most recent fragmentation time of that m/z in the prior MS2 schedule, not an earlier or later one

## Limitations

- During the first t0 = 15 s after fragmentation the weight is 0, so ions inside the standard exclusion window can never enter the priority list regardless of intensity
- Requires a prior TopN MS1/MS2 fragmentation schedule to derive tf per m/z — it cannot run on raw full-scan data alone
- Parameters (t0 = 15 s, t1 = 120 s) were tuned for the serum (QCA) and beer (QCB) extract experiments; other sample types or instruments may need re-optimization, e.g., via ViMMS simulation

## Evidence

- [methods] For each MS1 scan and each ion above the minimum MS1 intensity threshold, determine the most recent fragmentation time tf of its m/z value from the prior MS2 schedule.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the weight w(t) with the generalized dynamic-exclusion parameters t0 = 15 s and t1 = 120 s: w = 0 while t - tf <= t0, then a linear increase from 0 at t0 to 1 at t1.: [evidence span withheld — anchor status: not_found]
- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS): "Second, we describe how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS)"
