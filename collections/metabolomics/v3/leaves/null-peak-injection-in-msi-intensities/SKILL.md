---
name: null-peak-injection-in-msi-intensities
description: Use when validating or illustrating how undetected MS peaks (nulls) corrupt
  the Mann-Whitney U p-value in ion filtering between two MSI ROIs, and why rMSIKeyIon
  discards nulls before testing.
when_to_use_negative:
- Input is real rMSIproc peak-matrix data rather than a controlled simulation designed
  to isolate the null-value effect
- The goal is ion discovery on a real LDI-MS image with more than two clusters, where
  the three-parameter filter (null percentage, Mann-Whitney U test, fold-change) must
  be applied in full
- The statistical test is not the Mann-Whitney U test and fold-change on histograms,
  since the null-segregation rationale is specific to those parameters
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: rMSIKeyIon
  role: Runs TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4
    to apply the Mann-Whitney U test under nulls-kept vs nulls-segregated conditions
  repo: devtools::install_github("LlucSF/rMSIKeyIon")
- name: R
  role: Language used to simulate the two normal distributions with injected nulls
    and to run the comparison experiment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:9e455e552d3b7d81344d31896ef2cdb0dd5a85695c08007ad95b46624e1a7936
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - rMSIKeyIon
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# null-peak-injection-in-msi-intensities

## Summary

Simulates ion intensity histograms for two MSI regions with injected null (undetected) values below the zeroThreshold = 2.5e-4 convention, then demonstrates that segregating nulls out of the distributions before the Mann-Whitney U test (via rMSIKeyIon::TestIonSelect) reverses a non-significant result (p ≈ 0.38) into an extremely significant one (p ≈ 1e-43).

## When to use

Use when validating or illustrating how undetected MS peaks (nulls) corrupt the Mann-Whitney U p-value in ion filtering between two MSI ROIs, and why rMSIKeyIon discards nulls before testing.

## When NOT to use

- Input is real rMSIproc peak-matrix data rather than a controlled simulation designed to isolate the null-value effect
- The goal is ion discovery on a real LDI-MS image with more than two clusters, where the three-parameter filter (null percentage, Mann-Whitney U test, fold-change) must be applied in full
- The statistical test is not the Mann-Whitney U test and fold-change on histograms, since the null-segregation rationale is specific to those parameters

## Inputs

- Simulated ion intensity matrix (PeakMtx) for one ion across two clusters (200 samples per cluster) with injected null values below zeroThreshold = 2.5e-4
- Two-level clustering vector matching the rMSIKeyIon::TestIonSelect clusters argument

## Outputs

- Two Mann-Whitney U p-values: nulls-kept (~0.38) and nulls-segregated (~1e-43)
- results.Rdata containing the 'ions' and 'data' lists returned by TestIonSelect for both runs

## How to apply

In R, simulate two clusters of 200 ion intensity samples each, drawn from normal distributions with different means, and inject a fraction of null (undetected) values below the zeroThreshold = 2.5e-4 convention. Assemble the intensities into a peak-matrix-like structure (PeakMtx) plus a two-level clustering vector matching the rMSIKeyIon::TestIonSelect interface. First run TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4 keeping the nulls inside the histograms and record the p-value (expected ~0.38, non-significant). Then segregate the null values out of the histograms before the Mann-Whitney U test, rerun TestIonSelect, and record the second p-value (expected on the order of 1e-43, matching the paper's reported contrast). Store both outcomes, including the returned 'ions' and 'data' lists, as results.Rdata.

## Related tools

- **rMSIKeyIon** (Runs TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4 to apply the Mann-Whitney U test under nulls-kept vs nulls-segregated conditions) — devtools::install_github("LlucSF/rMSIKeyIon")
- **R** (Language used to simulate the two normal distributions with injected nulls and to run the comparison experiment)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Nulls-kept run yields a non-significant Mann-Whitney U p-value near 0.38
- Nulls-segregated run yields an extremely significant p-value of order 1e-43
- Each TestIonSelect call returns a list with an 'ions' list (up/down-regulated ions per comparison) and a 'data' list
- Injected null values all fall below the zeroThreshold = 2.5e-4 convention
- results.Rdata loads and contains both runs' outcomes

## Limitations

- The elevated number of undetected MS ions in a high percentage of pixels undermines reliability of statistical tests, which is exactly the effect this simulation exposes
- The simulated p-value magnitudes (0.38 vs 1e-43) depend on the chosen normal means and null fraction; different simulation parameters will not reproduce the paper's exact contrast
- The paper document lacks a discussion section, so limitations, missing artifacts, and reproducibility gaps cannot be further traced

## Evidence

- [abstract] Nulls discarded before test: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [intro] Motivation: many undetected ions: "Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels."
- [readme] TestIonSelect invocation: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
- [readme] Output structure: "The first one, called "ions" contains the up/down-regulated ions found at each comparison. The second one, called "data" contains all the data"
