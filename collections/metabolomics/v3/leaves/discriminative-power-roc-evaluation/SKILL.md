---
name: discriminative-power-roc-evaluation
description: Use when two reproducibility-detection methods must be compared on power/discrimination
  over simulated rank lists — e.g., MaRR vs. the Li et al. copula mixture model —
  with empirical FDR checked at alpha = 0.05.
when_to_use_negative:
- Real replicate datasets with no ground-truth labels of reproducible genes — ROC-based
  power comparison requires known truth, which simulation provides but empirical data
  does not
- When the analysis goal is raw-scale power or p-value-based power analysis rather
  than rank-scale discrimination — MaRR operates on maximum rank statistics Mg, so
  ROC must be on the rank scale
- When the task is only FDR calibration of a single method with no competing method
  to rank against — an ROC comparison of two procedures is not needed
tools:
- name: NumPy
  role: Generate the 100 simulated datasets per setting, drawing tg,1 ~ Unif(1,5)
    and the linearly degrading correlation structure
- name: SciPy
  role: Compute rank statistics and survival-function based quantities used in the
    ROC/power comparison
- name: pandas
  role: Aggregate per-setting, per-method FDR and power results and write simulation_b_fdr_power_results.csv
- name: matplotlib
  role: Plot ROC curves on the rank scale and per-setting FDR/power comparisons (e.g.,
    reproducing Figure 7-style output)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:4c4a3144e206ac56fb994403cd4f927f6741e77a88ee84f07825f2f25f3516e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - SciPy
  - matplotlib
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# discriminative-power-roc-evaluation

## Summary

Evaluates and compares the discriminative power of reproducibility-detection procedures (e.g., MaRR vs. the copula mixture model of Li et al. 2011) via ROC curves computed on the rank scale, alongside empirical FDR, across a grid of simulation settings.

## When to use

Use when two reproducibility-detection methods must be compared on power/discrimination over simulated rank lists — e.g., MaRR vs. the Li et al. copula mixture model — with empirical FDR checked at alpha = 0.05.

## When NOT to use

- Real replicate datasets with no ground-truth labels of reproducible genes — ROC-based power comparison requires known truth, which simulation provides but empirical data does not
- When the analysis goal is raw-scale power or p-value-based power analysis rather than rank-scale discrimination — MaRR operates on maximum rank statistics Mg, so ROC must be on the rank scale
- When the task is only FDR calibration of a single method with no competing method to rank against — an ROC comparison of two procedures is not needed

## Inputs

- Simulated bivariate gene-level test statistics (n=10,000 genes per dataset; 100 datasets per setting) with known reproducible/irreproducible gene labels
- Simulation setting grid: pi1 in {0.10, 0.30, 0.65, 0.80} crossed with r0 in {0.30, 0.70, 0.95}
- Per-method reproducible-gene calls (MaRR N-hat threshold; copula mixture model posterior calls)

## Outputs

- ROC curves (discriminative power) computed on the rank scale, one per method per setting
- Per-setting, per-method empirical FDR values aggregated over 100 datasets
- simulation_b_fdr_power_results.csv with per-setting, per-method empirical FDR and discriminative power

## How to apply

Generate replicate test statistic pairs per setting (Simulation B: pi1 in {0.10, 0.30, 0.65, 0.80}, r0 in {0.30, 0.70, 0.95}, 100 datasets of n=10,000 genes each) where rg degrades linearly with tg,1 from perfect correlation at tg,1=5 down to r0 at tg,1=1, drawing irreproducible signals from N(0, I). Convert test statistics to ranks (largest values most highly ranked), compute the maximum rank statistic Mg = max{Rgx, Rgy}, and apply each method's calls — MaRR's N-hat = max{k-hat < i <= n : mFDR-hat(i) <= alpha} with alpha = 0.05, and the copula mixture model run 10 times per dataset with random initializations, retaining the highest-likelihood fit. Compute empirical FDR per method per dataset as false calls divided by total genes declared reproducible, and compute discriminative power as ROC curves on the rank scale, exploiting the known ground-truth status of reproducible genes in simulation. Aggregate over the 100 datasets for each of the 12 settings and compare power curves per setting, verifying the copula model's conservatism where expected.

## Related tools

- **NumPy** (Generate the 100 simulated datasets per setting, drawing tg,1 ~ Unif(1,5) and the linearly degrading correlation structure)
- **SciPy** (Compute rank statistics and survival-function based quantities used in the ROC/power comparison)
- **pandas** (Aggregate per-setting, per-method FDR and power results and write simulation_b_fdr_power_results.csv)
- **matplotlib** (Plot ROC curves on the rank scale and per-setting FDR/power comparisons (e.g., reproducing Figure 7-style output))

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['pi1','r0','method','empirical_fdr','discriminative_power']).to_csv('simulation_b_fdr_power_results.csv', index=False)
```

## Evaluation signals

- MaRR must effectively control FDR at or below the nominal 0.05 level across all 12 settings
- MaRR must show higher discriminative power than the copula mixture model at r0 = 0.95 and pi1 = 0.80 (Simulation B, RNA-seq-like setting)
- The copula mixture model must be more conservative than MaRR when r0 = 0.95, matching reported Simulation B behavior in Figure 7
- simulation_b_fdr_power_results.csv contains rows for all 12 settings x 2 methods (24 method-setting cells), each aggregated over 100 datasets
- The copula mixture model result per dataset must come from the highest-likelihood fit among 10 random-initialization runs (muA ~ U(2,3), rho ~ U(0.70,0.99), pi1 +/- 0.10)

## Limitations

- Discriminative power via ROC requires known ground-truth reproducible-gene labels, so this evaluation is only well-defined in simulation, not on real replicate data
- The copula mixture model is initialization-sensitive: 10 random starts per dataset are required, retaining the maximum-likelihood fit, adding compute cost and residual optimizer variance
- Conclusions are tied to the Simulation B dependence model where correlation degrades linearly with tg,1 (rg = (1-r0)/4 * (tg,1-1) + r0); other dependence structures may not transfer

## Evidence

- [results] MaRR has higher discriminative power than the copula mixture model in Simulation B settings resembling RNA-seq data: [evidence span withheld — anchor status: not_found]
- [methods] Aggregate results over the 100 datasets for each of the 12 settings: [evidence span withheld — anchor status: not_found]
- [abstract] Our results on simulations and SEQC data show that the MaRR procedure effectively controls false discovery rates: "Our results on simulations and SEQC data show that the MaRR procedure effectively controls false discovery rates, has desirable power properties, and compares well to existing methods."
