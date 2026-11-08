---
name: lcms-feature-list-perturbation
description: 'Use when validating or calibrating an LC-HRMS feature-matching/alignment
  tool where ground truth is unknown: perturb a processed reference feature list (m/z
  + retention time) with known shifts and score recovery (F-measure, ppm discrepancy).'
when_to_use_negative:
- The input is already an aligned, combined data matrix rather than a reference feature
  list needing perturbation
- You have real replicate measurements with genuine retention-time drift, so no artificial
  transformation is required to obtain paired ground truth
- The goal is raw-profile data processing (peak picking/deconvolution) rather than
  perturbing and matching already-processed feature lists
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetMatch
  role: Semi-automated alignment tool whose m/z-offset detection and feature matching
    are evaluated against the perturbed feature lists; requires Java JRE 1.8 or newer
  repo: https://github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment (JRE 1.8+) required to execute the MetMatch alignment
    on the perturbed feature lists
- name: LC-HRMS
  role: Analytical data source providing the processed chromatograms (m/z values and
    retention times) that form the reference feature list to be perturbed
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:5c9c487716fb2086cd58bf86ec2d6003421a8455d39c82a00fa618b68e2387c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java
  - Java JRE
  - LC-HRMS
  - MetMatch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lcms-feature-list-perturbation

## Summary

Benchmark alignment software by artificially perturbing a reference LC-HRMS feature list with imposed m/z offsets and retention-time transformations plus random jitter, then measuring recovery accuracy. Grounded in MetMatch's validation approach of ±10 ppm offsets in 1 ppm steps and six RT-shift simulations.

## When to use

Use when validating or calibrating an LC-HRMS feature-matching/alignment tool where ground truth is unknown: perturb a processed reference feature list (m/z + retention time) with known shifts and score recovery (F-measure, ppm discrepancy).

## When NOT to use

- The input is already an aligned, combined data matrix rather than a reference feature list needing perturbation
- You have real replicate measurements with genuine retention-time drift, so no artificial transformation is required to obtain paired ground truth
- The goal is raw-profile data processing (peak picking/deconvolution) rather than perturbing and matching already-processed feature lists

## Inputs

- Processed LC-HRMS reference feature list (m/z values, retention times, peak areas)
- List of target perturbation parameters (e.g. m/z offsets -10 to +10 ppm in 1 ppm steps; RT transformation functions)
- Jitter specification (random ±6 s on retention times)

## Outputs

- Per-offset table of imposed vs detected m/z discrepancies (detected minus imposed, in ppm)
- Per-simulation counts of correctly matched peaks (Tp) and incorrectly matched peaks (Fp)
- Per-simulation F-measure values (e.g. written to a CSV such as metmatch_benchmark_results.csv)

## How to apply

Load a processed reference feature list (m/z values and retention times from LC-HRMS chromatograms) and generate modified copies. For m/z testing, create 21 copies shifted by -10 to +10 ppm in 1 ppm increments, run the tool's offset detection using histogram bins of 0.01 ppm, and tabulate imposed vs detected offsets. For retention-time testing, apply the target transformation functions (including discontinuous shifts) to the reference list, superimpose a random ±6 s jitter on all retention times, and optionally simulate alternative ion adducts of the same metabolite to mimic differently formed ions. Score matching against the known ground-truth correspondence by counting true positives (Tp) and false positives (Fp) and computing F-measures, e.g. F = 2*Tp/(2*Tp+Fp), expecting values near the published benchmark of 0.988.

## Related tools

- **MetMatch** (Semi-automated alignment tool whose m/z-offset detection and feature matching are evaluated against the perturbed feature lists; requires Java JRE 1.8 or newer) — https://github.com/S-T-K/MetMatch
- **Java** (Runtime environment (JRE 1.8+) required to execute the MetMatch alignment on the perturbed feature lists)
- **LC-HRMS** (Analytical data source providing the processed chromatograms (m/z values and retention times) that form the reference feature list to be perturbed)

## Evaluation signals

- All 21 detected m/z offsets must lie within +0.01 to -0.05 ppm of the artificially imposed values, with no correlation between offset magnitude and accuracy
- Per-simulation true positives fall within the reported range Tp 1389-1448 and false positives within Fp 5-16
- F-measures match reported values: 0.962 for the discontinuous RT-shift simulation (simulation 3) and 0.983-0.987 for the other five simulations versus the published benchmark of 0.988
- The tabulated results reproduce the Figure 8 table of imposed vs detected offsets when histogram bins of 0.01 ppm are used for offset detection

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer
- Random ±6 s jitter and the choice of discontinuous vs continuous RT transformations make per-run counts sensitive to the random seed; validation uses ranges (Tp 1389-1448) rather than exact values
- Simulated alternative adduct species approximate, but do not replace, real differently formed ion species observed across experiments

## Evidence

- [results] For all 21 simulated m/z offsets, the difference between the artificially imposed offset and the offset detected by MetMatch was within the range of +0.01 to −0.05 ppm: [evidence span withheld — anchor status: not_found]
- [results] 1389 peaks were correctly matched versus 16 incorrectly matched (F-measure 0.962): [evidence span withheld — anchor status: not_found]
- [intro] MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments"
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
