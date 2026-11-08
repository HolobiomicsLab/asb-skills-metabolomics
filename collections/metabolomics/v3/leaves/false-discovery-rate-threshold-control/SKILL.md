---
name: false-discovery-rate-threshold-control
description: Use when estimating reproducible metabolites (or features per sample
  pair) from replicate-rank data and you need the rejection region threshold chosen
  so empirical FDR is controlled at a specified alpha (e.g., 0.05).
when_to_use_negative:
- When the goal is a comprehensive evaluation of pre-processing steps (imputation/normalization)
  rather than using MaRR as a benchmark for comparison
- When the proportion of reproducible metabolites is small — the paper notes MaRR
  has lower power for smaller values of the reproducible proportion
- When reproducible and irreproducible components strongly overlap, since real signals
  may fall in undetectable regions
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: marr
  role: Implements the maximum rank reproducibility procedure including Marr() with
    alpha for FDR control, threshold arguments pSamplepairs/pFeatures, and accessor/plot
    functions for reproducibility outputs
  repo: http://bioconductor.org/packages/marr
- name: R
  role: Runtime for sourcing source_MaRR.R functions, running simulations (SIM_B1.R
    etc.), and saving result objects like B1_sim_results.Rdata
- name: devtools
  role: Used to install the marr package and MSPrep from GitHub for the reproducibility
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04336-9
    title: Reproducibility of mass spectrometry based metabolomics data
  source_content_hash: sha256:7a7677fb7ede97c284d5b13bb9d459add4d0fe51b650e7c49681592c154c50aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - marr
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# false-discovery-rate-threshold-control

## Summary

A skill for controlling the false discovery rate at a nominal level alpha when selecting a data-driven rejection threshold N-hat from maximum rank reproducibility (MaRR) statistics, so that only reproducible metabolites/sample pairs are called.

## When to use

Use when estimating reproducible metabolites (or features per sample pair) from replicate-rank data and you need the rejection region threshold chosen so empirical FDR is controlled at a specified alpha (e.g., 0.05).

## When NOT to use

- When the goal is a comprehensive evaluation of pre-processing steps (imputation/normalization) rather than using MaRR as a benchmark for comparison
- When the proportion of reproducible metabolites is small — the paper notes MaRR has lower power for smaller values of the reproducible proportion
- When reproducible and irreproducible components strongly overlap, since real signals may fall in undetectable regions

## Inputs

- Metabolite (feature) by sample matrix (data frame, matrix, or SummarizedExperiment with one assay) from MS-metabolomics replicate experiments
- Nominal FDR level alpha (default 0.05)
- Reproducibility thresholds pSamplepairs and pFeatures (default 0.75)
- MaRR tuning parameter lambda (0.9) for the pi_1 estimator

## Outputs

- Estimated reproducible proportion pi_1-hat of metabolites
- mFDR curve mFDR-hat(l) across candidate rejection regions (0, l)
- Rejection threshold N-hat at nominal FDR level alpha
- Percent reproducible features per sample pair and sample pairs per feature, plus filtered reproducible calls at thresholds (c_s, c_m)

## How to apply

Compute the maximal rank statistic for each replicate sample pair and the empirical survival function of ranks from the metabolite-by-sample matrix. Estimate the reproducible proportion pi_1 via the MSE(lambda)-based estimator with tuning parameter lambda = 0.9, then evaluate the mFDR estimate mFDR-hat(l) = (l-k-hat)^2/(Q(l)(M-k-hat)) for each candidate rejection region (0, l). Select the rejection threshold N-hat at the nominal FDR level alpha (Marr() default alpha = 0.05), and flag reproducible metabolites/sample pairs using the data-dependent thresholds (c_s, c_m) after inspecting reproducibility histograms. The nonparametric rationale avoids parametric assumptions on the distributions or dependence structures of reproducible metabolites.

## Related tools

- **marr** (Implements the maximum rank reproducibility procedure including Marr() with alpha for FDR control, threshold arguments pSamplepairs/pFeatures, and accessor/plot functions for reproducibility outputs) — http://bioconductor.org/packages/marr
- **R** (Runtime for sourcing source_MaRR.R functions, running simulations (SIM_B1.R etc.), and saving result objects like B1_sim_results.Rdata)
- **devtools** (Used to install the marr package and MSPrep from GitHub for the reproducibility pipeline)

## Examples

```
MarrOutput <- Marr(object = dataSE, pSamplepairs = 0.75, pFeatures = 0.75, alpha = 0.05)
```

## Evaluation signals

- Bias (estimated minus true reproducible proportion) is close to zero in simulations (e.g., SIM_B1 with true pi_1 = 0.35)
- Empirical FDR at the selected threshold N-hat is controlled at the nominal level alpha
- mFDR-hat(l) = (l-k-hat)^2/(Q(l)(M-k-hat)) is monotonically well-defined over candidate rejection regions
- Reproducibility histograms (MarrPlotSamplepairs/MarrPlotFeatures) support the chosen thresholds c_s, c_m
- Saved R objects (e.g., B1_sim_results.Rdata) reproduce simulation design outputs per the paper's SIM/SIMPLOT script convention

## Limitations

- MaRR has lower power for detecting reproducible metabolites when the proportion of reproducible metabolites is small
- Lack of separation between reproducible and irreproducible components means true signals may fall in undetectable regions
- No comprehensive evaluation of pre-processing steps was performed; the approach only benchmarks comparisons

## Evidence

- [methods] mFDR estimator definition: [evidence span withheld — anchor status: not_found]
- [abstract] FDR control under realistic settings: "the MaRR procedure effectively controls the False Discovery Rate (FDR) when there was a gradual reduction in correlation between replicate pairs for less highly ranked signals"
- [abstract] Bias close to zero: "Bias (i.e., the difference between the estimated and the true value of reproducible signal proportions) values for simulations are also close to zero"
- [readme] Marr function parameters: [evidence span withheld — anchor status: external_doc]
- [discussion] Data-dependent thresholds with histogram visualization: [evidence span withheld — anchor status: not_found]
