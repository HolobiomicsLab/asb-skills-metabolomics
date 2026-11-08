---
name: f-measure-calculation
description: Use when evaluating how accurately an LC-HRMS alignment tool (e.g. MetMatch)
  matches peaks to a reference, from known true-positive and false-positive counts
  against a ground-truth correspondence.
when_to_use_negative:
- When the goal is to correct or align the LC-HRMS data itself rather than to quantify
  how well an alignment already performed — use MetMatch's m/z and retention-time
  shift correction instead.
- When no ground-truth correspondence between the perturbed and reference feature
  lists exists, since Tp/Fp counts cannot be derived without it.
- When the evaluation target is a different metric, e.g. mass accuracy in ppm or retention-time
  residual distribution, rather than matching accuracy.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetMatch
  role: Produces the aligned match lists on which the F-measure is computed, by matching
    corresponding ions and combining differently formed ion species of the same metabolite
    into a single data matrix.
  repo: https://github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to execute the MetMatch alignment tool whose
    match results feed the F-measure calculation.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:90dd4ab379d537a6e12336ca1186d799d32db899ecaeb6d6a7eb1827331c0a60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/S-T-K/MetMatch
  tools:
  - Java
  - MetMatch
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# f-measure-calculation

## Summary

Computes the F-measure (harmonic mean of precision and recall) to quantify alignment accuracy of LC-HRMS feature matching, as done to validate MetMatch against simulated retention-time shift and adduct-formation perturbations.

## When to use

Use when evaluating how accurately an LC-HRMS alignment tool (e.g. MetMatch) matches peaks to a reference, from known true-positive and false-positive counts against a ground-truth correspondence.

## When NOT to use

- When the goal is to correct or align the LC-HRMS data itself rather than to quantify how well an alignment already performed — use MetMatch's m/z and retention-time shift correction instead.
- When no ground-truth correspondence between the perturbed and reference feature lists exists, since Tp/Fp counts cannot be derived without it.
- When the evaluation target is a different metric, e.g. mass accuracy in ppm or retention-time residual distribution, rather than matching accuracy.

## Inputs

- Aligned match list (matched feature pairs between the perturbed and reference LC-HRMS feature lists, with matched m/z, retention time and peak-area entries)
- Ground-truth correspondence between the perturbed feature list and the reference feature list (feature indices or identifiers designating which peaks are the same metabolite/ion)
- Per-simulation true-positive count (Tp) and false-positive count (Fp) — e.g. from the metmatch_benchmark_results.csv comparison step

## Outputs

- Per-simulation F-measure value (e.g. 0.962 for the discontinuous shift simulation, 0.983–0.987 for continuous shift simulations)
- Per-simulation Tp and Fp counts written to metmatch_benchmark_results.csv
- Benchmark comparison value (0.988 for the un-perturbed alignment) for judging whether the perturbed runs remain within reported ranges

## How to apply

Run the alignment tool on a perturbed feature list whose true correspondence to the reference feature list is known, then count true positives (Tp) and false positives (Fp) among the returned matches. Compute the F-measure as the harmonic mean of precision (Tp / (Tp + Fp)) and recall (Tp / (Tp + Fn)), where Fn is derived from the known ground-truth correspondence. In the MetMatch evaluation this was applied to six artificial retention-time shift simulations (including a discontinuous transformation, simulation 3) superimposed with a random ±6 s jitter and simulated alternative ion adducts, and benchmarked against the un-perturbed alignment (benchmark F-measure 0.988). Accept the evaluation if per-simulation Tp and Fp fall within expected ranges (Tp 1389–1448, Fp 5–16) and the F-measure matches the reported values (0.962 for simulation 3; 0.983–0.987 for the remaining simulations).

## Related tools

- **MetMatch** (Produces the aligned match lists on which the F-measure is computed, by matching corresponding ions and combining differently formed ion species of the same metabolite into a single data matrix.) — https://github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to execute the MetMatch alignment tool whose match results feed the F-measure calculation.)

## Examples

```
python scripts/evaluate_alignment.py --perturbed sim3_features.csv --reference ref_features.csv --groundtruth truth.csv --metric fmeasure --out metmatch_benchmark_results.csv
```

## Evaluation signals

- Per-simulation true-positive count must fall in the reported range (Tp 1389–1448) and false-positive count in the range (Fp 5–16).
- F-measure for the discontinuous retention-time shift simulation (simulation 3) should be 0.962 (1389 correct vs. 16 incorrect matches).
- F-measure for the remaining continuous shift simulations should be in the range 0.983–0.987, versus the benchmark of 0.988 obtained without artificial shifts.
- The per-simulation Tp, Fp, and F-measure values must be written to metmatch_benchmark_results.csv with one row per simulation.
- The benchmark run (no artificial shifts, no jitter, no adduct simulation) should yield F-measure 0.988; deviations indicate that the evaluation setup, not the metric, is at fault.

## Limitations

- The F-measure is only as good as the ground-truth correspondence: in the MetMatch evaluation the ground truth was known because the perturbations were artificially applied to the reference feature list, so the metric cannot be computed the same way for real unlabelled samples.
- The reported ranges (Tp 1389–1448, Fp 5–16) and F-measure values are specific to the six simulated shift transformations and the ±6 s jitter used in this study; other perturbation magnitudes or datasets will produce different values.
- MetMatch requires Java runtime (JRE) version 1.8 or newer, and is distributed under the GNU General Public License without any warranty — evaluation scripts built on its output inherit those constraints.

## Evidence

- [abstract] MetMatch recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: [evidence span withheld — anchor status: not_found]
- [abstract] Corresponding ions and metabolites are matched and combined into a single data matrix: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.: "MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE."
