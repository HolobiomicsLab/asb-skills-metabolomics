---
name: mass-spectrometry-dda-scoring-reimplementation
description: 'Use when reimplementing or evaluating DDA precursor-prioritization scoring:
  MS1 survey-scan ions with known prior fragmentation times (tf) need nonbinary dynamic-exclusion
  weights and a top-N MS2 target list per scan.'
when_to_use_negative:
- Input is DIA or SWATH data rather than DDA MS1 survey scans — there are no prior
  MS2 fragmentation times tf to weight against
- No prior MS2 acquisition schedule exists for the sample, so tf cannot be determined
  per m/z (e.g., first-pass analysis with no exclusion history)
- Input is already an aggregated feature table without time-resolved scan-level intensities,
  since w(t) is defined per ion observed at time t relative to tf
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype, implement, optimize,
    and validate the WeightedDEW scoring strategy in silico before real-instrument
    deployment
- name: MZmine 2
  role: Source of the full-scan mzML peak bounding boxes loaded as input to the weight
    computation
- name: XCMS
  role: Peak-picking/feature toolchain supplying the survey-scan ion detections that
    get weighted and ranked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:0220bbffa1931ee906cc56f540ac233aee5f2fae0a8f430f82a37fe0147496da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectrometry-dda-scoring-reimplementation

## Summary

Implements the WeightedDEW ion-prioritization weight w(t), which generalizes the standard dynamic exclusion window into a nonbinary score, and emits a per-scan MS2 fragmentation priority ranking. This is the concrete scoring function behind the paper's improved DDA strategies developed and validated in silico with ViMMS.

## When to use

Use when reimplementing or evaluating DDA precursor-prioritization scoring: MS1 survey-scan ions with known prior fragmentation times (tf) need nonbinary dynamic-exclusion weights and a top-N MS2 target list per scan.

## When NOT to use

- Input is DIA or SWATH data rather than DDA MS1 survey scans — there are no prior MS2 fragmentation times tf to weight against
- No prior MS2 acquisition schedule exists for the sample, so tf cannot be determined per m/z (e.g., first-pass analysis with no exclusion history)
- Input is already an aggregated feature table without time-resolved scan-level intensities, since w(t) is defined per ion observed at time t relative to tf

## Inputs

- Full-scan mzML file with MS1 peak bounding boxes (serum QCA / beer QCB extracts)
- TopN-derived MS1/MS2 scan schedule defining prior fragmentation times tf per m/z
- Minimum MS1 intensity threshold and dynamic-exclusion parameters (t0 = 15 s, t1 = 120 s)

## Outputs

- Per-MS1-scan ranking table with a weight column w in [0, 1] for every ranked ion
- Top-N WeightedDEW MS2 fragmentation priority list per scan

## How to apply

Load the full-scan mzML peak bounding boxes and the TopN-derived MS1/MS2 scan schedule (serum QCA and beer QCB extracts in the paper). For each MS1 scan, determine for every ion above the minimum MS1 intensity threshold the most recent fragmentation time tf of its m/z from the prior MS2 schedule. Compute w(t) with the generalized dynamic-exclusion parameters t0 = 15 s and t1 = 120 s: w = 0 while t - tf <= t0, then a linear rise from 0 at t0 to 1 at t1. Multiply each ion's survey-scan intensity by its weight and rank ions in descending weighted-intensity order, selecting the top N per MS1 scan as the WeightedDEW MS2 priority list. The rationale is that standard DDA wastes fragmentation effort on recently fragmented ions; the linear ramp lets partially excluded ions re-enter selection smoothly rather than switching on/off at a fixed exclusion cutoff.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype, implement, optimize, and validate the WeightedDEW scoring strategy in silico before real-instrument deployment)
- **MZmine 2** (Source of the full-scan mzML peak bounding boxes loaded as input to the weight computation)
- **XCMS** (Peak-picking/feature toolchain supplying the survey-scan ion detections that get weighted and ranked)

## Examples

```
w = 0.0 if (t - tf) <= 15 else min((t - tf - 15) / (120 - 15), 1.0); ranked = sorted(ions, key=lambda i: i.intensity * w(i.t, i.tf), reverse=True)[:N]
```

## Evaluation signals

- Every ranked ion carries a weight column with values in [0, 1]
- w = 0 exactly while t - tf <= t0 = 15 s (standard exclusion applies)
- w increases linearly from 0 at t0 = 15 s to 1 at t1 = 120 s, matching the linear-rise behavior described in the paper's Figure 3
- Per-scan output is a descending weighted-intensity (intensity x w) ordering with exactly the top N ions selected as the MS2 priority list
- t0 = 15 s and t1 = 120 s parameters are applied consistently across all MS1 scans

## Limitations

- The scoring depends on tf, the most recent fragmentation time of each m/z from the prior MS2 schedule; without accurate TopN-derived schedule information the weights are undefined
- Exclusion parameters t0 = 15 s and t1 = 120 s were tuned for the paper's serum (QCA) and beer (QCB) extracts and may need re-optimization for other chromatography or instrument settings
- DDA by nature only fragments a small subset of the ions present, so even weighted prioritization cannot guarantee coverage of all eluting ions in a single injection

## Evidence

- [other] For each ion observed at time t, the weight w is defined relative to tf, the most recent fragmentation time of that m/z: a standard exclusion applies for the first t0 seconds after fragmentation, after which the weight increases linearly from 0 at t0 to 1 at t1.: [evidence span withheld — anchor status: not_found]
- [other] Compute the weight w(t) with the generalized dynamic-exclusion parameters t0 = 15 s and t1 = 120 s: w = 0 while t - tf <= t0, then a linear increase from 0 at t0 to 1 at t1.: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract on the two improved DDA methods: "development and optimization of two new DDA methods that introduce new advanced ion prioritization strategies."
- [intro] Article intro on the in silico prototyping workflow that grounds this skill: "how new strategies can be prototyped, implemented, optimized, and validated using a virtual metabolomics mass spectrometer (ViMMS),10 reducing the traditional need for a large"
- [intro] Article intro on DDA coverage limitation: "Here, we address the problem of improving DDA coverage for a single injection"
