---
name: alignment-benchmark-evaluation
description: Use when a new or modified LC-HRMS alignment configuration needs quantitative
  validation against a ground-truth benchmark, e.g. after changing m/z or retention-time
  tolerance settings or adduct-recombination rules.
when_to_use_negative:
- The input is already an aligned feature matrix rather than unaligned LC-HRMS feature
  lists requiring retention-time and m/z shift correction
- The analysis goal is peak detection or feature finding rather than evaluating alignment
  of already-processed chromatograms
- No ground-truth (or artificially simulated) feature correspondences exist, so Tp/Fp
  and F-measure cannot be computed
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMatch
  role: Semi-automated alignment tool whose m/z and retention-time tolerance settings
    are benchmarked; matches corresponding ions and recombines adduct variants
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to execute MetMatch (JRE version 1.8 or newer)
- name: LC-HRMS
  role: Analytical data source providing the processed feature lists used in the alignment
    benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:8000cb64d13391ec6622d0ee715e8a23a1250c1ac6b387b6bf8602727885b0d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - LC-HRMS
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# alignment-benchmark-evaluation

## Summary

Benchmark an LC-HRMS feature alignment tool (MetMatch) by applying artificial m/z and retention-time transformations to a reference feature list and scoring matches against known ground-truth correspondences using true positives, false positives, and F-measure.

## When to use

Use when a new or modified LC-HRMS alignment configuration needs quantitative validation against a ground-truth benchmark, e.g. after changing m/z or retention-time tolerance settings or adduct-recombination rules.

## When NOT to use

- The input is already an aligned feature matrix rather than unaligned LC-HRMS feature lists requiring retention-time and m/z shift correction
- The analysis goal is peak detection or feature finding rather than evaluating alignment of already-processed chromatograms
- No ground-truth (or artificially simulated) feature correspondences exist, so Tp/Fp and F-measure cannot be computed

## Inputs

- Reference LC-HRMS feature list with m/z values, retention times, and chromatographic peak areas
- Retention-time transformation functions (including the discontinuous simulation-3 transformation) with random ±6 s jitter
- Simulated alternative ion adduct species for a subset of features
- Ground-truth correspondence between perturbed and reference features

## Outputs

- Per-simulation true positive (Tp) and false positive (Fp) counts (e.g. metmatch_benchmark_results.csv)
- Per-simulation F-measure values (0.962 for simulation 3; 0.983–0.987 for the other five simulations)
- Aligned/combined data matrix of matched ions with chromatographic peak areas, m/z values and retention times

## How to apply

Load the reference LC-HRMS feature list (m/z values, retention times, peak areas) and apply known retention-time transformation functions, including a discontinuous transformation, superimposing a random ±6 s jitter on all retention times. Generate simulated alternative ion adduct species for a subset of features so the tool must account for differently formed ion species of the same metabolite. Run the MetMatch alignment (Java-based, JRE 1.8 or newer) of each perturbed feature list against the reference, using its m/z and retention-time tolerance settings to match corresponding ions and recombine adduct variants. Count true positives (Tp) and false positives (Fp) per simulation against the known ground truth, compute the F-measure, and compare with the published benchmark F-measure of 0.988. In the reported test, six shift simulations (including the discontinuous simulation 3) yielded F-measures of 0.962–0.987, with Tp of 1389–1448 and Fp of 5–16.

## Related tools

- **MetMatch** (Semi-automated alignment tool whose m/z and retention-time tolerance settings are benchmarked; matches corresponding ions and recombines adduct variants) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to execute MetMatch (JRE version 1.8 or newer))
- **LC-HRMS** (Analytical data source providing the processed feature lists used in the alignment benchmark)

## Evaluation signals

- Per-simulation Tp falls within the reported range 1389–1448 and Fp within 5–16
- F-measure for the discontinuous retention-time shift simulation (simulation 3) equals the reported 0.962
- F-measures for the remaining five simulations fall in the reported 0.983–0.987 range versus the benchmark of 0.988
- Random ±6 s retention-time jitter is reproducibly superimposed on all retention times of the perturbed lists
- Matched ions and adduct variants are combined into a single data matrix containing peak areas, m/z values and retention times

## Limitations

- The discontinuous retention-time transformation (simulation 3) is the hardest case and yields the lowest F-measure (0.962)
- MetMatch requires Java runtime (JRE) version 1.8 or newer
- MetMatch is distributed without any warranty under the GNU General Public License (version 2 or later)
- Discussion-section content is not available in the provided document, so deeper caveats could not be traced

## Evidence

- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [abstract] Semi-automated alignment of different measurement sequences based on a reference feature list: [evidence span withheld — anchor status: not_found]
