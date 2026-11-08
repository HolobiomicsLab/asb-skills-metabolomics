---
name: dynamic-exclusion-weight-computation
description: 'Use when simulating or implementing WeightedDEW ion prioritization for
  DDA LC-MS/MS: converting MS1 survey-scan intensities plus prior MS2 fragmentation
  times into a per-ion nonbinary exclusion weight and a per-scan priority ranking.'
when_to_use_negative:
- When the goal is to replay or evaluate standard (binary) TopN DDA with a fixed exclusion
  window rather than the WeightedDEW generalization
- When the input lacks a prior MS2 fragmentation schedule, so tf per m/z cannot be
  determined and w(t) is undefined
- When analysis targets DIA acquisition rather than DDA ion prioritization — the article
  notes the DDA vs DIA comparison remains inconclusive and this skill addresses DDA
  coverage
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype, optimize, and validate
    the WeightedDEW weighting and ranking in silico before hardware deployment
- name: MZmine 2
  role: Provides the TopN-derived MS1/MS2 scan schedule and peak information for serum
    (QCA) and beer (QCB) extracts from which tf is derived
- name: XCMS
  role: Used to derive the peak/scan schedule underlying the exclusion-time computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:1dcdf69a269a17d242bdc2d71cfc53dd77fb5c3f4cca169623a0371d93a569cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dynamic-exclusion-weight-computation

## Summary

Implements the WeightedDEW controller from Davies et al. (Anal. Chem. 2021), which generalizes the standard dynamic exclusion window into a nonbinary weight w in [0, 1] per ion, used to rank MS1 ions for MS2 fragmentation priority in DDA acquisition.

## When to use

Use when simulating or implementing WeightedDEW ion prioritization for DDA LC-MS/MS: converting MS1 survey-scan intensities plus prior MS2 fragmentation times into a per-ion nonbinary exclusion weight and a per-scan priority ranking.

## When NOT to use

- When the goal is to replay or evaluate standard (binary) TopN DDA with a fixed exclusion window rather than the WeightedDEW generalization
- When the input lacks a prior MS2 fragmentation schedule, so tf per m/z cannot be determined and w(t) is undefined
- When analysis targets DIA acquisition rather than DDA ion prioritization — the article notes the DDA vs DIA comparison remains inconclusive and this skill addresses DDA coverage

## Inputs

- Full-scan mzML file with peak bounding boxes (MS1 survey scans)
- TopN-derived MS1/MS2 scan schedule (fragmentation history giving tf per m/z)
- Dynamic-exclusion parameters t0 = 15 s and t1 = 120 s
- Minimum MS1 intensity threshold and TopN count N

## Outputs

- Per-scan WeightedDEW priority table: ion m/z, intensity, weight w in [0, 1], weighted intensity, descending rank
- Top-N MS2 fragmentation priority list per MS1 scan

## How to apply

Load the full-scan mzML peak bounding boxes together with the TopN-derived MS1/MS2 scan schedule (e.g. serum QCA and beer QCB extracts processed with MZmine 2/XCMS). For each MS1 scan and each ion above the minimum MS1 intensity threshold, determine tf, the most recent fragmentation time of that m/z from the prior MS2 schedule. Compute w(t) with the generalized dynamic-exclusion parameters t0 = 15 s and t1 = 120 s: w = 0 while t - tf <= t0, then a linear rise from 0 at t0 to 1 at t1. Multiply each ion's survey-scan intensity by its weight and rank ions in descending weighted-intensity order, selecting the top N per MS1 scan as the WeightedDEW fragmentation priority list. The rationale is that standard DDA with a binary exclusion window fragments only a small subset of ions, whereas the linear ramp allows still-eluting ions to re-enter fragmentation earlier, increasing unique-fragment coverage in a single injection.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype, optimize, and validate the WeightedDEW weighting and ranking in silico before hardware deployment)
- **MZmine 2** (Provides the TopN-derived MS1/MS2 scan schedule and peak information for serum (QCA) and beer (QCB) extracts from which tf is derived)
- **XCMS** (Used to derive the peak/scan schedule underlying the exclusion-time computation)

## Examples

```
w = 0.0 if (t - tf) <= t0 else min((t - tf - t0) / (t1 - t0), 1.0)  # t0=15 s, t1=120 s; rank by intensity * w descending, take top N
```

## Evaluation signals

- Every ranked ion has a weight column w with values within [0, 1]
- w = 0 exactly while t - tf <= t0 (first 15 s after fragmentation) for all ions
- w increases linearly from 0 at t0 = 15 s to 1 at t1 = 120 s, matching the linear-rise behavior of Figure 3
- The t0 = 15 s and t1 = 120 s parameters are applied consistently across all MS1 scans in the output
- Per-scan ranking table is ordered by descending intensity-weighted values and truncated to the top N ions

## Limitations

- Weights depend entirely on the correctness of tf (most recent fragmentation time of that m/z); errors in the MS2 schedule propagate into the ranking
- Parameter sensitivity: the linear ramp is defined only between t0 = 15 s and t1 = 120 s; ions fragmented more than t1 seconds ago receive full weight and are treated as unexcluded
- As a DDA strategy it still fragments only a subset of ions present, so coverage gains are bounded relative to the theoretically optimal performance computed for the injection

## Evidence

- [full_text] the weight w is defined relative to tf, the most recent fragmentation time of that m/z: a standard exclusion applies for the first t0 seconds after fragmentation: [evidence span withheld — anchor status: not_found]
- [other] Compute the weight w(t) with the generalized dynamic-exclusion parameters t0 = 15 s and t1 = 120 s: w = 0 while t - tf <= t0, then a linear increase from 0 at t0 to 1 at t1.: [evidence span withheld — anchor status: not_found]
- [abstract] an in silico framework for fast and cost-efficient development of new DDA strategies: "an in silico framework for fast and cost-efficient development of new DDA strategies"
- [abstract] development and optimization of two new DDA methods that introduce new advanced ion prioritization strategies: "development and optimization of two new DDA methods that introduce new advanced ion prioritization strategies"
