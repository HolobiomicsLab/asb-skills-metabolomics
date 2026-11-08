---
name: reversed-phase-liquid-chromatography-data-comparison
description: Use when you have paired RPLC datasets (two analytical runs or configurations
  sharing detected compounds) with retention order data and configuration metadata
  (eluent pH, organic-phase modifier/percentage, temperature), and need to identify
  which chromatographic factor drives retention-order.
when_to_use_negative:
- The datasets are HILIC rather than RPLC — the paired retention-order replicability
  analysis here applies to the reversed-phase system (HILIC generalizability is stated
  as future work in the article)
- The goal is predicting retention scores or ranking candidate structures for identification
  (use ROASMI itself) rather than assessing cross-method replicability of existing
  experimental retention orders
- The input is a single dataset with no chromatographically paired configuration to
  compare against — no CCRS can be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Source repository supplying the 59 paired RPLC retention datasets and code
    used to recreate the manuscript's retention-replicability analysis
  repo: https://github.com/FangYuan717/ROASMI
- name: scipy
  role: Computes Pearson r and p-values between ΔpH (or ΔF) and CCRS with 95% confidence
    intervals
- name: matplotlib
  role: Renders the CCRS vs ΔpH / ΔF scatter panels with confidence intervals
- name: seaborn
  role: Renders violin plots of CCRS grouped by organic-phase modifier and other configuration
    attributes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:5e034b650753120e336e047d40a1889173af180c568ca1efdacc2137693170bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - matplotlib
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# reversed-phase-liquid-chromatography-data-comparison

## Summary

A paired-dataset comparison workflow that quantifies how well retention orders replicate across RPLC configurations using the CCRS (correlation coefficient of retention sequences), then attributes differences to chromatographic factors — chiefly eluent pH (ΔpH), organic-phase percentage (ΔF), modifier type, and column temperature.

## When to use

Use when you have paired RPLC datasets (two analytical runs or configurations sharing detected compounds) with retention order data and configuration metadata (eluent pH, organic-phase modifier/percentage, temperature), and need to identify which chromatographic factor drives retention-order replicability.

## When NOT to use

- The datasets are HILIC rather than RPLC — the paired retention-order replicability analysis here applies to the reversed-phase system (HILIC generalizability is stated as future work in the article)
- The goal is predicting retention scores or ranking candidate structures for identification (use ROASMI itself) rather than assessing cross-method replicability of existing experimental retention orders
- The input is a single dataset with no chromatographically paired configuration to compare against — no CCRS can be computed

## Inputs

- Paired RPLC retention datasets (59 pairs collated from RepoRT, PredRet, Metabolome Workbench, MetaboLights), e.g. ST001683, dataset_56, dataset_61
- Per-compound retention order (rank) data for compounds shared between each dataset pair
- Chromatographic configuration attributes per dataset: eluent pH, organic-phase percentage and modifier type, column temperature

## Outputs

- Per-pair table of CCRS values and pair attributes exported to results/ccrs_per_pair.csv
- Composite scatter/violin figure (e.g. figures/fig4_a_to_h.png) with Pearson r and 95% CI annotations
- Mean CCRS and 95% CI for the ΔpH ≤ 1 subset

## How to apply

Load the 59 paired RPLC retention datasets (collated from RepoRT, PredRet, Metabolome Workbench, and MetaboLights sources). For each dataset pair, compute the CCRS between paired retention orders (rank sequences) of the shared compounds only. For the same pairs compute the configuration differences — ΔpH, ΔF (organic-phase percentage difference), and column temperature difference — then regress CCRS against each factor, reporting Pearson r with a 95% confidence interval (scipy.stats.pearsonr with matplotlib scatter, e.g. Fig. 4a-b for ΔpH and Fig. 4g-h for ΔF). Stratify by ΔpH ≤ 1 to report mean CCRS with 95% CI (Fig. 4c), and use seaborn violin plots of CCRS grouped by organic-phase modifier, Δtemperature, and other configuration attributes (Fig. 4d-f) to compare categorical factors. The rationale is that RTs alone replicate poorly across chromatographic methods, so ordinal agreement (retention sequence) is the replicability metric, and the factor with the strongest CCRS association identifies the dominant driver — in this analysis, eluent pH.

## Related tools

- **ROASMI** (Source repository supplying the 59 paired RPLC retention datasets and code used to recreate the manuscript's retention-replicability analysis) — https://github.com/FangYuan717/ROASMI
- **scipy** (Computes Pearson r and p-values between ΔpH (or ΔF) and CCRS with 95% confidence intervals)
- **matplotlib** (Renders the CCRS vs ΔpH / ΔF scatter panels with confidence intervals)
- **seaborn** (Renders violin plots of CCRS grouped by organic-phase modifier and other configuration attributes)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(pairs['delta_pH'], pairs['CCRS'])
```

## Evaluation signals

- Every dataset pair has a CCRS computed only over compounds shared between the two configurations, and all pairs appear in results/ccrs_per_pair.csv with their ΔpH, ΔF, and Δtemperature attributes
- Pearson r and 95% confidence interval are reported for both the ΔpH vs CCRS and ΔF vs CCRS regressions
- The ΔpH ≤ 1 subset summary reports a mean CCRS with a 95% CI, and the ΔpH association is stronger than ΔF, reproducing the article's finding that eluent pH is the main driver of retention-order replicability
- Violin plots group CCRS by categorical configuration attributes (organic-phase modifier, column temperature difference) without mixing RPLC and non-RPLC data
- Figure outputs (fig4_a_to_h.png) contain all eight panels matching the paper's Fig. 4a-h layout

## Limitations

- When chromatographic peaks of two compounds are close enough together, their retention order may not be consistent across experiments, adding noise to CCRS
- RT outliers resulting from slight changes in experimental conditions remain unresolved and require mechanistic study
- The analysis addresses the reversed-phase system only; constructing retention models with generalizability under the HILIC system is left as future work

## Evidence

- [intro] The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs).: "The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs)."
- [intro] The distribution of differences in experimental RTs observed between different configurations (source data: ST001683 dataset [10]): "The distribution of differences in experimental RTs observed between different configurations (source data: ST001683 dataset [10])"
- [intro] Nonetheless, determining the degree of ChrSS required for replicable retention orders remains to be explored.: "Nonetheless, determining the degree of ChrSS required for replicable retention orders remains to be explored."
- [readme] In addition to code and trained models, this repository also contains data that allows researchers to recreate the entire analysis for the manuscript: "In addition to code and trained models, this repository also contains data that allows researchers to recreate the entire analysis for the manuscript"
