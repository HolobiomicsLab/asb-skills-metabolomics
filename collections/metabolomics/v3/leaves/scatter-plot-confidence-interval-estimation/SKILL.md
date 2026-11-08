---
name: scatter-plot-confidence-interval-estimation
description: Use when you have paired chromatographic datasets with a per-pair agreement
  score (e.g. CCRS) and a continuous per-pair difference attribute (ΔpH, ΔF, Δtemperature),
  and you need to test and visualize whether that attribute explains replicability.
when_to_use_negative:
- The grouping variable is categorical (organic-phase modifier, polarity, column type)
  rather than a continuous difference — a grouped violin plot of CCRS (Fig. 4d-f style)
  is the appropriate comparison, not a Pearson correlation.
- The data are HILIC or non-RPLC retention pairs — ROASMI and its CCRS analysis apply
  only to the RPLC system (eluent pH around 2.7), so association estimates would be
  out of scope.
- Individual compound pairs have very close chromatographic peaks whose elution order
  flips across experiments — these retention-order inconsistencies add noise that
  should be excluded or flagged before correlation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scipy
  role: scipy.stats.pearsonr computes the Pearson correlation coefficient between
    ΔpH (or ΔF) and CCRS with its 95% confidence interval
- name: matplotlib
  role: renders the ΔpH vs CCRS and ΔF vs CCRS scatter plots annotated with r and
    CI
- name: seaborn
  role: seaborn violinplot summarizes CCRS distributions grouped by categorical configuration
    attributes (organic-phase modifier, temperature difference)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:77e9a1818d9d104655409a8ac4fd91c217a9d6909857bd2a86702ecd9a4f045a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - matplotlib
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scatter-plot-confidence-interval-estimation

## Summary

Quantifies the association between a continuous chromatographic factor (ΔpH or ΔF) and retention-order replicability (CCRS) across paired RPLC datasets, reporting Pearson r with a 95% confidence interval on a scatter plot. This is how the ROASMI paper established that eluent pH is the main driver of retention order replicability.

## When to use

Use when you have paired chromatographic datasets with a per-pair agreement score (e.g. CCRS) and a continuous per-pair difference attribute (ΔpH, ΔF, Δtemperature), and you need to test and visualize whether that attribute explains replicability.

## When NOT to use

- The grouping variable is categorical (organic-phase modifier, polarity, column type) rather than a continuous difference — a grouped violin plot of CCRS (Fig. 4d-f style) is the appropriate comparison, not a Pearson correlation.
- The data are HILIC or non-RPLC retention pairs — ROASMI and its CCRS analysis apply only to the RPLC system (eluent pH around 2.7), so association estimates would be out of scope.
- Individual compound pairs have very close chromatographic peaks whose elution order flips across experiments — these retention-order inconsistencies add noise that should be excluded or flagged before correlation.

## Inputs

- 59 paired RPLC retention datasets (RepoRT, PredRet, Metabolome Workbench, MetaboLights collated in the ROASMI paper data folder)
- Per-pair table of CCRS values and chromatographic configuration attributes (ΔpH, ΔF, Δ column temperature, organic-phase modifier)

## Outputs

- Scatter plots of ΔpH vs CCRS (Fig. 4a-b) and ΔF vs CCRS (Fig. 4g-h) with Pearson r and 95% CI
- ΔpH≤1 subset summary of mean CCRS with 95% CI (Fig. 4c)
- Violin plots of CCRS grouped by organic-phase modifier, temperature difference and other configuration attributes (Fig. 4d-f)
- results/ccrs_per_pair.csv and figures/fig4_a_to_h.png

## How to apply

For each of the 59 paired RPLC retention datasets (from RepoRT, PredRet, Metabolome Workbench and MetaboLights sources), compute the CCRS — the correlation coefficient of retention sequences over shared compounds — and the continuous differences ΔpH, ΔF (organic-phase percentage difference) and Δ column temperature. Plot ΔpH vs CCRS (Fig. 4a-b) and ΔF vs CCRS (Fig. 4g-h) as scatter plots annotated with the Pearson r from scipy.stats.pearsonr and its 95% confidence interval. For the high-similarity regime, restrict to the ΔpH≤1 subset and report the mean CCRS with its 95% CI (Fig. 4c). For categorical configuration attributes (organic-phase modifier, temperature difference), use grouped seaborn violin plots of CCRS (Fig. 4d-f) rather than a correlation. Export per-pair CCRS values and pair attributes to results/ccrs_per_pair.csv and the composite figure to figures/fig4_a_to_h.png.

## Related tools

- **scipy** (scipy.stats.pearsonr computes the Pearson correlation coefficient between ΔpH (or ΔF) and CCRS with its 95% confidence interval)
- **matplotlib** (renders the ΔpH vs CCRS and ΔF vs CCRS scatter plots annotated with r and CI)
- **seaborn** (seaborn violinplot summarizes CCRS distributions grouped by categorical configuration attributes (organic-phase modifier, temperature difference))

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(pairs.delta_pH, pairs.ccrs)  # 95% CI via bootstrap over the 59 dataset pairs
```

## Evaluation signals

- results/ccrs_per_pair.csv exists and contains one row per dataset pair with CCRS plus pair attributes (ΔpH, ΔF, Δ temperature, configuration)
- Each scatter panel reports a Pearson r with a 95% confidence interval computed via scipy.stats.pearsonr
- The ΔpH≤1 subset (Fig. 4c) reports mean CCRS with a 95% CI, isolating the high-replicability regime
- Composite figure figures/fig4_a_to_h.png reproduces panels a–h (scatters, subset summary, violin plots)
- Directionality check: CCRS should decrease as ΔpH increases, consistent with eluent pH being the main driver of retention-order replicability

## Limitations

- Retention order of very close chromatographic peaks may not be consistent across experiments, attenuating CCRS and weakening the correlation
- RT outliers resulting from slight changes in experimental conditions are unresolved and require further mechanistic study, so the ΔpH–CCRS relationship is not perfectly deterministic
- The analysis is correlational over RPLC dataset pairs within the paper's collated sources; it does not establish causality and does not extend to HILIC systems

## Evidence

- [other] workflow scatter with CI: [evidence span withheld — anchor status: not_found]
- [results] main finding: [evidence span withheld — anchor status: not_found]
- [readme] readme scope: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around"
- [abstract] abstract generalizability: "The generalizability of ROASMI is proven by 71 independent reversed-phase liquid chromatography (RPLC) datasets."
