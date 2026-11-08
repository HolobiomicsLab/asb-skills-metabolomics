---
name: adduct-species-simulation
description: Use when you need to stress-test an LC-HRMS feature alignment tool (e.g.
  MetMatch) against a known ground truth by injecting artificial adduct variants and
  retention-time perturbations into a reference feature list.
when_to_use_negative:
- The input is already an aligned, merged data matrix — MetMatch's role is to combine
  corresponding ions and metabolites from unaligned feature lists into a single data
  matrix
- You need to detect adducts in real untargeted metabolomics samples rather than benchmark
  alignment behavior with artificially simulated adduct species
- Retention-time shifts are unknown and uncontrolled — this skill requires a defined
  transformation (e.g. the discontinuous simulation 3 with ±6 s jitter) to establish
  ground truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetMatch
  role: Java-based semi-automated alignment tool that matches corresponding ions across
    perturbed and reference feature lists using m/z and retention-time tolerances
    and recombines adduct variants
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment (JRE 1.8 or newer) required to execute the MetMatch alignment
    step
- name: LC-HRMS
  role: Analytical data modality providing the m/z, retention-time, and peak-area
    feature lists that are perturbed and aligned
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:add9aceff951df78e2691d7e6d5217748f0a85ce2fff80905d4c3796b9eb03be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - LC-HRMS
  - MetMatch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-species-simulation

## Summary

Generates simulated alternative ion adduct species for a subset of LC-HRMS features so that MetMatch's ability to recombine differently formed ion species of the same metabolite during alignment can be benchmarked against ground truth, using F-measures over six retention-time shift simulations.

## When to use

Use when you need to stress-test an LC-HRMS feature alignment tool (e.g. MetMatch) against a known ground truth by injecting artificial adduct variants and retention-time perturbations into a reference feature list.

## When NOT to use

- The input is already an aligned, merged data matrix — MetMatch's role is to combine corresponding ions and metabolites from unaligned feature lists into a single data matrix
- You need to detect adducts in real untargeted metabolomics samples rather than benchmark alignment behavior with artificially simulated adduct species
- Retention-time shifts are unknown and uncontrolled — this skill requires a defined transformation (e.g. the discontinuous simulation 3 with ±6 s jitter) to establish ground truth

## Inputs

- Reference LC-HRMS feature list with m/z values, retention times, and chromatographic peak areas
- Retention-time transformation specification (six functions from Section 3.10.3, including the discontinuous simulation 3) plus random ±6 s jitter
- Subset of features selected to receive simulated alternative adduct species
- Ground-truth feature correspondence mapping for scoring

## Outputs

- Six perturbed LC-HRMS feature lists (one per retention-time shift simulation) containing simulated adduct variants
- metmatch_benchmark_results.csv with per-simulation true positives (Tp), false positives (Fp), and F-measure values

## How to apply

Load a reference LC-HRMS feature list (m/z values, retention times, peak areas) into the MetMatch workflow, then apply the six retention-time transformation functions of Section 3.10.3 — including the discontinuous transformation of simulation 3 — superimposing a random ±6 s jitter on all retention times. Generate simulated alternative adduct species for a subset of features to emulate differently formed ions of the same metabolite, mirroring MetMatch's design goal of accounting for differently formed ion species. Run the Java-based MetMatch alignment of each perturbed feature list against the reference, using its m/z and retention-time tolerance settings to match corresponding ions and recombine adduct variants. Compare aligned matches against the known ground-truth correspondences to count true positives (Tp) and false positives (Fp) per simulation, and compute the F-measure. Write per-simulation Tp, Fp, and F-measure values to metmatch_benchmark_results.csv.

## Related tools

- **MetMatch** (Java-based semi-automated alignment tool that matches corresponding ions across perturbed and reference feature lists using m/z and retention-time tolerances and recombines adduct variants) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment (JRE 1.8 or newer) required to execute the MetMatch alignment step)
- **LC-HRMS** (Analytical data modality providing the m/z, retention-time, and peak-area feature lists that are perturbed and aligned)

## Evaluation signals

- Per-simulation true positives fall within the reported range Tp 1389–1448 and false positives within Fp 5–16
- F-measure for the discontinuous retention-time shift simulation (simulation 3) equals 0.962 (1389 correctly matched vs 16 incorrectly matched peaks)
- F-measures for the remaining five simulations fall in the 0.983–0.987 range versus the published benchmark of 0.988
- Each perturbed feature list retains the simulated ±6 s retention-time jitter and the simulated adduct variants for the chosen feature subset
- metmatch_benchmark_results.csv contains one Tp, Fp, and F-measure row for each of the six simulations

## Limitations

- Ground truth exists only because retention-time shifts and adduct species are artificially simulated; results may not transfer to uncontrolled real-world measurement-sequence drift
- The benchmark F-measure (0.988) is slightly above the simulation results (0.962–0.987), so perfect recovery is not expected even under controlled perturbation
- MetMatch requires Java runtime (JRE) version 1.8 or newer, and it is distributed WITHOUT ANY WARRANTY, as a free GNU GPL-licensed tool

## Evidence

- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [abstract] Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix.: "Corresponding ions and metabolites are matched and chromatographic peak areas, m/z values and retention times are combined into a single data matrix."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
