---
name: imputation-runtime-benchmarking
description: Use when comparing the computational cost of imputation algorithms (KNN-EU
  vs KNN-CR vs KNN-TN) on an identical metabolomics dataset, or when quantifying the
  runtime penalty of truncated-normal MLE estimation before choosing an imputer.
when_to_use_negative:
- The goal is imputation accuracy (RMSE, MLCI, concordance of significance testing)
  rather than computational cost — this skill measures runtime only.
- Metabolite distributions are strongly non-normal, since KNN-TN's truncated-normal
  MLE assumption is what generates the extra runtime and its validity.
- Datasets have excessive missingness or very small samples, where truncated mean/variance
  estimation is unstable and timings would reflect degenerate fits rather than typical
  cost.
tools:
- name: Newton-Raphson optimization
  role: Numerical optimizer used to fit the truncated-normal MLE of the mean and standard
    deviation in KNN-TN, with sample estimates as starting values; this fitting step
    is the source of KNN-TN's runtime overhead (19.95 s vs 1.81–3.41 s).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:db4d7e6d48f99c9a357fd7a0ca38ab27564138161af0620f239ca2f2b9fcfaae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Newton-Raphson optimization
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# imputation-runtime-benchmarking

## Summary

Benchmarks wall-clock runtime of metabolomics KNN imputers (KNN-EU, KNN-CR, KNN-TN) on a fixed simulated dataset with MNAR+MAR missingness, quantifying the overhead of truncated-normal MLE mean/SD estimation via Newton–Raphson in KNN-TN.

## When to use

Use when comparing the computational cost of imputation algorithms (KNN-EU vs KNN-CR vs KNN-TN) on an identical metabolomics dataset, or when quantifying the runtime penalty of truncated-normal MLE estimation before choosing an imputer.

## When NOT to use

- The goal is imputation accuracy (RMSE, MLCI, concordance of significance testing) rather than computational cost — this skill measures runtime only.
- Metabolite distributions are strongly non-normal, since KNN-TN's truncated-normal MLE assumption is what generates the extra runtime and its validity.
- Datasets have excessive missingness or very small samples, where truncated mean/variance estimation is unstable and timings would reflect degenerate fits rather than typical cost.

## Inputs

- Simulated 50-sample × 400-metabolite complete matrix from a multivariate normal distribution (metabolite means Uniform(−5, 5); blockwise ρw=0.7/ρoff=0.2, AR(1) ρ=0.9, or mixed correlation structure)
- Missingness specification: 15% total (10% below LOD at the 10% complete-data quantile as MNAR + 5% removed at random as MAR)
- Imputer parameter K=10 nearest neighbors for all three methods (KNN-EU, KNN-CR, KNN-TN)

## Outputs

- runtime_benchmark.csv with per-method single-run wall-clock times in seconds (reference: KNN-EU 1.81 s, KNN-CR 3.41 s, KNN-TN 19.95 s)
- Average total runtime in minutes across the three missing-mechanism settings (reference: ≈1.20 min)

## How to apply

Simulate a 50-sample × 400-metabolite dataset from a multivariate normal distribution with metabolite means drawn from Uniform(−5, 5) and a fixed correlation structure (blockwise ρw=0.7/ρoff=0.2, AR(1) with ρ=0.9, or mixed blockwise), so all methods are timed on identical data. Impose 15% total missingness with MNAR exceeding MAR (e.g., 10% of values below the LOD, defined as the 10% quantile of the complete data, plus 5% removed at random), then drop metabolites with more than 75% missing observations. Implement the three imputers with K=10 neighbors: KNN-EU (Euclidean distance, unstandardized), KNN-CR (correlation distance d_C = 1 − |r| with sample mean/SD standardization), and KNN-TN (correlation distance using truncated-normal MLE mean/SD fitted by Newton–Raphson with sample estimates as starting values, estimated only for metabolites whose sample mean lies within three SDs of the LOD). Wrap each method in a wall-clock timer, run once per method, record seconds, and repeat across the three missing-mechanism settings to get average total runtime in minutes; write results to runtime_benchmark.csv. Interpret ordering: KNN-TN is expected to be slowest because of the truncated-normal mean/SD estimation step, not the neighbor search itself.

## Related tools

- **Newton-Raphson optimization** (Numerical optimizer used to fit the truncated-normal MLE of the mean and standard deviation in KNN-TN, with sample estimates as starting values; this fitting step is the source of KNN-TN's runtime overhead (19.95 s vs 1.81–3.41 s).)

## Examples

```
python benchmark_runtime.py --n-samples 50 --n-metabolites 400 --missing 0.15 --k 10 --out runtime_benchmark.csv
```

## Evaluation signals

- Per-method single-run times should be of the same order as the reference values: KNN-EU ≈1.81 s, KNN-CR ≈3.41 s, KNN-TN ≈19.95 s on a 50×400 dataset at 15% missingness.
- Ordering invariant: KNN-EU < KNN-CR < KNN-TN in runtime, since KNN-TN additionally estimates truncated means and standard deviations.
- Average total runtime across the three missing-mechanism settings should be near ≈1.20 min for the 50-sample/400-metabolite configuration.
- All three methods must be timed on the identical simulated dataset and missingness pattern, with timers scoped to imputation only.
- runtime_benchmark.csv must contain per-method seconds and average total runtime in minutes with no missing runtime entries.

## Limitations

- KNN-TN's runtime cost is driven by truncated-normal MLE estimation (Newton–Raphson with sample starting values, applied only to metabolites whose sample mean lies within three SDs of the LOD); timings scale with the number of eligible metabolites.
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN.
- The KNN-TN likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated.
- Single-run timings are noisy; the reference workflow averages over the three missing-mechanism settings and compares against published per-run values.

## Evidence

- [abstract] we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods: "we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods"
- [discussion] we introduce an extension to the KNN imputation algorithm which handles truncated data, termed KNN-TN: "we introduce an extension to the KNN imputation algorithm which handles truncated data, termed KNN-TN"
- [discussion] the cleaning process (which removes metabolites with >75% MVs): "the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR"
