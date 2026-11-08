---
name: response-factor-error-binning-analysis
description: Use when evaluating accuracy of inferred absolute concentration scaling
  factors (response factors) from relative abundance GC-MS time-course data, by binning
  per-metabolite errors against known ground truth and comparing to negative controls.
when_to_use_negative:
- Data are noisy real measurements without simulated ground-truth response factors
  — no true RF reference exists to bin errors against.
- The goal is de novo inference on a new metabolic system rather than benchmarking
  MetaboPAC against negative controls on simulated data.
- Assessing noisy-data performance across sampling frequencies/CoV requires the MetaboPAC_*_noisy.m
  variant workflow, not the noiseless error-binning setup.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetaboPAC
  role: Source of inferred response factors being benchmarked via error-bin fractions
    against trueRF values
- name: MATLAB
  role: Runtime environment for MetaboPAC_*.m scripts, data generation, and aggregation
    of error-bin fractions
- name: GC-MS
  role: Measurement modality whose relative-abundance data the response factors convert
    to absolute concentrations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1039/d2mo00168c
    title: Towards inferring absolute concentrations from relative abundance in time-course
      GC-MS metabolomics data
  source_content_hash: sha256:5a80af6a073d372dc2ef65e4cc04db5c9c9de1f695db9da4c8b71a63f15d93ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GC-MS
  - MATLAB
  - MetaboPAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# response-factor-error-binning-analysis

## Summary

Benchmarking analysis that compares MetaboPAC-inferred response factors to true simulated response factors by computing the fraction falling within error tolerance bins of log2(1.1), log2(1.3), and log2(1.5), then statistically testing against negative-control methods.

## When to use

Use when evaluating accuracy of inferred absolute concentration scaling factors (response factors) from relative abundance GC-MS time-course data, by binning per-metabolite errors against known ground truth and comparing to negative controls.

## When NOT to use

- Data are noisy real measurements without simulated ground-truth response factors — no true RF reference exists to bin errors against.
- The goal is de novo inference on a new metabolic system rather than benchmarking MetaboPAC against negative controls on simulated data.
- Assessing noisy-data performance across sampling frequencies/CoV requires the MetaboPAC_*_noisy.m variant workflow, not the noiseless error-binning setup.

## Inputs

- MetaboPAC-inferred response factors per repetition and percKnownKinetics value
- True response factors from trueRF folder (20 replicate sets per system, uniform or log-uniform between 1 and 1000)
- Negative-control comparator response-factor inference results
- Model/system identifiers for the four metabolic models

## Outputs

- Per-bin fractions of inferred response factors within log2(1.1), log2(1.3), log2(1.5) error tolerances
- Aggregated fractions table across repetitions and percKnownKinetics levels (heuristic_noiseless_rf_error_bins.csv)
- Statistical comparison results of MetaboPAC vs. negative controls at each percKnownKinetics level

## How to apply

For each of the 20 repetitions and each percKnownKinetics value (0, 20, 40, 60, 80, 100), take MetaboPAC's inferred response factors and compare them element-wise to the true response factors (e.g., the 20 replicate sets in the trueRF folder, sampled uniformly between 1 and 1000, or log-uniform for the *_log variant). Classify each metabolite's inference error into tolerance bins of log2(1.1), log2(1.3), and log2(1.5), recording the fraction of response factors falling within each bin per run. Aggregate bin fractions across the 20 repetitions for each metabolic model and each percKnownKinetics level, alongside corresponding negative-control comparator results, then statistically compare MetaboPAC's bin fractions against the negative controls at each percKnownKinetics level. Write aggregated fractions to a table such as heuristic_noiseless_rf_error_bins.csv; on noiseless data, significant improvement over negative controls is expected to emerge when at least 20% of kinetic terms are known a priori.

## Related tools

- **MetaboPAC** (Source of inferred response factors being benchmarked via error-bin fractions against trueRF values)
- **MATLAB** (Runtime environment for MetaboPAC_*.m scripts, data generation, and aggregation of error-bin fractions)
- **GC-MS** (Measurement modality whose relative-abundance data the response factors convert to absolute concentrations)

## Examples

```
Run MetaboPAC_*.m for repetitions [1:20] and percKnownKinetics values of [0 20 40 60 80 100], then aggregate log2 error-bin fractions and write heuristic_noiseless_rf_error_bins.csv.
```

## Evaluation signals

- Fraction of inferred response factors within log2(1.5) tolerance should increase as percKnownKinetics rises from 0 to 100.
- Reproduces Figure S5–S6 trend: heuristic approach bin fractions significantly exceed negative controls on noiseless data.
- Significant improvement over negative controls should appear once at least 20% of kinetic terms are known a priori.
- Aggregated CSV (heuristic_noiseless_rf_error_bins.csv) should contain bin fractions aggregated over exactly 20 repetitions for each of the four models and six percKnownKinetics values.
- True RF sets reproduce the trueRF folder structure: 20 replicate sets per system.

## Limitations

- MetaboPAC only applies to intracellular metabolomics samples in a single compartment and assumes all metabolites are directly connected via reactions.
- Response factors were assumed constant with concentration; linear dynamic range or ion suppression may violate this assumption.
- Multiple sets of response factors can have similar or smaller penalty values than the true ones, suggesting penalties need refinement.
- Prediction accuracy varies widely (40% to nearly 100%) depending on network, data quality, and error tolerance.

## Evidence

- [methods] compare inferred response factors to the true response factors and compute the fraction falling within error tolerances of log2(1.1), log2(1.3), and log2(1.5): [evidence span withheld — anchor status: not_found]
- [methods] validation trend: [evidence span withheld — anchor status: not_found]
