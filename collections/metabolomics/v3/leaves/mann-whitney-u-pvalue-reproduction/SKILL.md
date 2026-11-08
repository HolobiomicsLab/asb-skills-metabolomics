---
name: mann-whitney-u-pvalue-reproduction
description: Use when validating or benchmarking how undetected MS peaks (null values)
  affect Mann-Whitney U p-values in ion filtering between two clusters of mass-specimaging
  intensities, e.g. testing rMSIKeyIon's TestIonSelect behavior.
when_to_use_negative:
- Input is real MSI peak-matrix data where null treatment has already been applied
  by TestIonSelect — the contrast being demonstrated requires raw histograms with
  retained nulls for the baseline run
- The analysis goal is ion selection between more than two ROIs or clusters rather
  than reproducing the two-cluster null-segregation p-value contrast
- You only need the demo outputs without rerunning the simulation — load the repository's
  results.Rdata instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: rMSIKeyIon
  role: Provides TestIonSelect, the ion-filtering function whose Mann-Whitney U p-values
    are compared with and without null-value segregation
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: R
  role: Runtime environment for the simulation, the two TestIonSelect runs, and storing
    results.Rdata
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:dcc215cd4f6602b9fd6900a4c84c9017e687e2cb176a0756dd53bb72672ba05e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - R
  - rMSIKeyIon
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mann-whitney-u-pvalue-reproduction

## Summary

Reproduces the rMSIKeyIon paper's demonstration that segregating undetected (null) MS peaks from ion histograms before the Mann-Whitney U test flips a non-significant result (~p=0.38) into an extremely significant one (~p=1e-43), justifying the package's null-treatment step.

## When to use

Use when validating or benchmarking how undetected MS peaks (null values) affect Mann-Whitney U p-values in ion filtering between two clusters of mass-specimaging intensities, e.g. testing rMSIKeyIon's TestIonSelect behavior.

## When NOT to use

- Input is real MSI peak-matrix data where null treatment has already been applied by TestIonSelect — the contrast being demonstrated requires raw histograms with retained nulls for the baseline run
- The analysis goal is ion selection between more than two ROIs or clusters rather than reproducing the two-cluster null-segregation p-value contrast
- You only need the demo outputs without rerunning the simulation — load the repository's results.Rdata instead

## Inputs

- Simulated ion intensity matrix (PeakMtx) with two clusters of 200 samples each from normal distributions with different means
- Fraction of injected null (undetected) values below zeroThreshold = 2.5e-4
- Two-level clustering vector compatible with rMSIKeyIon::TestIonSelect

## Outputs

- Mann-Whitney U p-value from the nulls-kept run (expected ~0.38)
- Mann-Whitney U p-value from the nulls-segregated run (expected ~1e-43)
- results.Rdata containing the TestIonSelect 'ions' and 'data' lists for both runs

## How to apply

In R, simulate two clusters of 200 ion intensity samples each, drawn from normal distributions with different means, injecting a fraction of null (undetected) values below the zeroThreshold = 2.5e-4 convention. Assemble the simulated intensities into a PeakMtx-like structure plus a two-level clustering vector matching the rMSIKeyIon::TestIonSelect interface. First run TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4 keeping nulls inside the histograms and record the Mann-Whitney U p-value (expected ~0.38). Then segregate the null values out of the histograms before the test, rerun, and record the second p-value (expected on the order of 1e-43); store both outcomes with the returned 'ions' and 'data' lists as results.Rdata. The rationale is the paper's claim that undetected MS signals (null peaks) are discarded from the histogram before calculating the Mann-Whitney U test and fold-change parameters because elevated null counts otherwise undermine test reliability.

## Related tools

- **rMSIKeyIon** (Provides TestIonSelect, the ion-filtering function whose Mann-Whitney U p-values are compared with and without null-value segregation) — https://github.com/LlucSF/rMSIKeyIon
- **R** (Runtime environment for the simulation, the two TestIonSelect runs, and storing results.Rdata)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Nulls-kept run yields a non-significant Mann-Whitney U p-value near 0.38
- Nulls-segregated run yields an extremely significant p-value on the order of 1e-43
- TestIonSelect returns a list with two sub-lists named 'ions' and 'data' for each comparison
- Both run outcomes persist correctly via load/save of results.Rdata

## Limitations

- High percentage of pixels with undetected MS ions makes statistical test reliability difficult, so results depend heavily on how many nulls are injected
- The package requires the GSL library (RcppGSL linkage), a known installation hurdle especially on Windows
- No discussion/changelog text is available, so reproducibility gaps and failure modes beyond the documented p-value contrast cannot be verified

## Evidence

- [abstract] Undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters.: "The undetected MS signals (null peaks) are discarded from the histogram before the calculation of (b) and (c) parameters."
- [methods] results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix,"
- [readme] It uses the classical p-value/ FC method and introduces a third novel method to treat undetected MS peaks more accurately.: "It uses the classical p-value/ FC method and introduces a third novel method to treat undetected MS peaks more accurately."
- [abstract] Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels.: "Another difficulty with the reliability of statistical tests is the elevated number of undetected MS ions in a high percentage of pixels."
