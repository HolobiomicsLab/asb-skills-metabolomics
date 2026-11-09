---
name: retention-order-correlation-analysis
description: Use when multiple paired RPLC retention datasets (e.g., from RepoRT,
  PredRet, Metabolome Workbench, MetaboLights) share compounds and you need to test
  whether eluent pH or other configuration attributes explain retention-order replicability.
when_to_use_negative:
- You need to predict retention orders for candidate molecules rather than measure
  replicability across existing datasets — use ROASMI (RankNet/D-MPNN model) instead.
- The datasets are HILIC or unpaired single-run data with no shared compounds between
  runs — CCRS between paired retention orders cannot be computed.
- You are scoring MS/MS spectra or integrating spectral scores; this skill analyzes
  only retention-order agreement, not MS/MS evidence.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Its repository (FangYuan717/ROASMI) provides the data folder with the paired
    RPLC retention datasets and configuration metadata needed to recreate the retention-replicability
    analysis
  repo: https://github.com/FangYuan717/ROASMI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:bf91a80fc9b50eb757d4e0fab3b99f7efa5c897339aedeac5386efa29fe95e74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# retention-order-correlation-analysis

## Summary

Quantifies how well compound elution order replicates between paired RPLC datasets using CCRS (correlation coefficient of retention sequences), then relates CCRS to chromatographic configuration differences (ΔpH, ΔF, Δtemperature) to identify which factor drives retention-order replicability.

## When to use

Use when multiple paired RPLC retention datasets (e.g., from RepoRT, PredRet, Metabolome Workbench, MetaboLights) share compounds and you need to test whether eluent pH or other configuration attributes explain retention-order replicability.

## When NOT to use

- You need to predict retention orders for candidate molecules rather than measure replicability across existing datasets — use ROASMI (RankNet/D-MPNN model) instead.
- The datasets are HILIC or unpaired single-run data with no shared compounds between runs — CCRS between paired retention orders cannot be computed.
- You are scoring MS/MS spectra or integrating spectral scores; this skill analyzes only retention-order agreement, not MS/MS evidence.

## Inputs

- 59 paired RPLC retention datasets (RepoRT, PredRet, Metabolome Workbench, MetaboLights sources) with per-compound retention times/orders
- Per-dataset chromatographic configuration metadata: eluent pH, organic-phase percentage (F) and modifier type, column temperature, other configuration attributes
- Mapping of compound identities shared between dataset pairs

## Outputs

- results/ccrs_per_pair.csv: per-pair CCRS values with pair attributes (ΔpH, ΔF, Δtemperature, configuration)
- figures/fig4_a_to_h.png: composite figure with ΔpH-vs-CCRS and ΔF-vs-CCRS scatter plots (Pearson r + 95% CI), ΔpH≤1 subset summary, and CCRS violin plots grouped by configuration attributes

## How to apply

Load the 59 paired RPLC retention datasets collated from the data folder of the ROASMI repository; for each dataset pair, restrict to shared compounds and compute CCRS, the correlation coefficient of the retention sequences. For each pair, compute ΔpH, ΔF (organic-phase percentage difference), Δ column temperature, and record chromatographic configuration attributes. Correlate each difference metric with CCRS using scipy.stats.pearsonr, reporting Pearson r with a 95% confidence interval (Fig. 4a-b for ΔpH, Fig. 4g-h for ΔF). For the ΔpH≤1 subset, report the mean CCRS and its 95% CI (Fig. 4c), and use seaborn violinplots to compare CCRS grouped by organic-phase modifier, temperature difference, and other configuration attributes (Fig. 4d-f). The rationale, per the article, is that RT replicability across chromatographic methods is the key barrier to using retention data in annotation, and the degree of chromatographic space similarity required for replicable orders was unexplored.

## Related tools

- **ROASMI** (Its repository (FangYuan717/ROASMI) provides the data folder with the paired RPLC retention datasets and configuration metadata needed to recreate the retention-replicability analysis) — https://github.com/FangYuan717/ROASMI

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['delta_pH'], df['CCRS'])
```

## Evaluation signals

- Every CCRS value in results/ccrs_per_pair.csv lies within [-1, 1], with CCRS near 1 indicating nearly identical retention sequences
- Number of rows in ccrs_per_pair.csv equals the number of dataset pairs sharing compounds, and each row carries non-missing ΔpH, ΔF, and Δtemperature attributes
- Pearson r and 95% CI are reported for both the ΔpH-vs-CCRS and ΔF-vs-CCRS regressions, with the pH correlation strongest (eluent pH is the paper's reported main factor)
- Mean CCRS of the ΔpH≤1 subset is reported with a 95% CI and is higher than the CCRS of pairs with larger ΔpH
- Composite figure figures/fig4_a_to_h.png contains all eight panels (a-h) including scatter plots and the grouped violin plots

## Limitations

- Retention order of very close chromatographic peaks may not be consistent across experiments, adding noise to CCRS for pairs of compounds with nearly identical RTs
- RT outliers resulting from slight changes in experimental conditions remain unresolved and would require mechanistic study
- The analysis covers RPLC systems only; HILIC retention replicability and generalizable HILIC modeling are stated future work

## Evidence

- [intro] The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs).: "The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs)."
- [intro] Nonetheless, determining the degree of ChrSS required for replicable retention orders remains to be explored.: "Nonetheless, determining the degree of ChrSS required for replicable retention orders remains to be explored."
- [discussion] when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments.: "when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments."
- [readme] We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around 2.7.: "We provide four initial ROASMI models (ROASMI_1 - ROASMI_5) for predicting the retention behavior of compounds in the reversed-phase liquid chromatography (RPLC) system with an eluent pH of around"
