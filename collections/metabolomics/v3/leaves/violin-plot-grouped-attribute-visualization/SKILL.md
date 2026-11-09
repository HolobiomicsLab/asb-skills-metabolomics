---
name: violin-plot-grouped-attribute-visualization
description: Use when comparing a distribution of pair-level replicability or similarity
  scores (e.g., CCRS between retention orders) across categorical chromatographic
  configuration attributes such as organic-phase modifier or column temperature difference.
when_to_use_negative:
- The comparison variable is continuous (e.g., ΔpH or ΔF vs CCRS) — use a scatter
  plot with Pearson r and 95% confidence interval instead of grouped violins
- Data are per-compound RTs rather than per-pair replicability scores; compute CCRS
  between paired retention orders of shared compounds first
- The system is HILIC rather than RPLC — the article's replicability analysis (and
  ROASMI) applies only to the RPLC system
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ROASMI
  role: Source of the CCRS replicability analysis pipeline and collated paired RPLC
    datasets used to build the grouped violin panels
  repo: https://github.com/FangYuan717/ROASMI
- name: seaborn
  role: seaborn.violinplot renders the CCRS distributions grouped by configuration
    attribute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:5554a9f349ec9188cf6619883a1608b811850d435ebdafe3d749b35c3aedca47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# violin-plot-grouped-attribute-visualization

## Summary

Generates violin plots of per-pair CCRS (correlation coefficient of retention sequences) grouped by chromatographic configuration attributes (e.g., organic-phase modifier, Δ column temperature) to visually assess which factors drive retention-order replicability across paired RPLC datasets.

## When to use

Use when comparing a distribution of pair-level replicability or similarity scores (e.g., CCRS between retention orders) across categorical chromatographic configuration attributes such as organic-phase modifier or column temperature difference.

## When NOT to use

- The comparison variable is continuous (e.g., ΔpH or ΔF vs CCRS) — use a scatter plot with Pearson r and 95% confidence interval instead of grouped violins
- Data are per-compound RTs rather than per-pair replicability scores; compute CCRS between paired retention orders of shared compounds first
- The system is HILIC rather than RPLC — the article's replicability analysis (and ROASMI) applies only to the RPLC system

## Inputs

- Per-pair table (results/ccrs_per_pair.csv) with CCRS values and chromatographic attributes (ΔpH, ΔF, Δ column temperature, organic-phase modifier) for 59 paired RPLC retention datasets (RepoRT, PredRet, Metabolome Workbench, MetaboLights sources)

## Outputs

- Violin plot panels (Fig. 4d–f) of CCRS grouped by organic-phase modifier, column temperature difference, and other configuration attributes
- Composite figure file figures/fig4_a_to_h.png combining scatter and violin panels

## How to apply

After computing per-pair CCRS values and pair attributes (ΔpH, ΔF organic-phase percentage difference, Δ column temperature, configuration attributes), build a grouped violin plot of CCRS distributions per attribute level using seaborn's violinplot. In the ROASMI paper this produced Fig. 4d–f, complementing the scatter plots of ΔpH vs CCRS and ΔF vs CCRS (with Pearson r and 95% CI) so that eluent pH could be identified as the main factor affecting retention-order replicability. Export the per-pair CCRS values and attributes to results/ccrs_per_pair.csv first, then render the composite figure to figures/fig4_a_to_h.png so the violin panels are reproducible alongside the quantitative statistics.

## Related tools

- **ROASMI** (Source of the CCRS replicability analysis pipeline and collated paired RPLC datasets used to build the grouped violin panels) — https://github.com/FangYuan717/ROASMI
- **seaborn** (seaborn.violinplot renders the CCRS distributions grouped by configuration attribute)

## Examples

```
import seaborn as sns; sns.violinplot(data=ccrs_per_pair, x='organic_phase_modifier', y='CCRS')
```

## Evaluation signals

- Each violin panel's x-axis levels correspond to the attribute values recorded in results/ccrs_per_pair.csv (e.g., organic-phase modifier categories), and y-axis CCRS values fall within the expected correlation range (approximately -1 to 1)
- The ΔpH≤1 subset statistics (mean CCRS and 95% CI) reported in Fig. 4c are consistent with the distributions shown in the violin panels
- The composite figure figures/fig4_a_to_h.png contains all eight panels (a–h) including the violin plots grouped by attribute
- Rerunning the script on the same 59 paired datasets reproduces the same violin shapes, confirming deterministic grouping from the CSV

## Limitations

- RTs are poorly replicable across different chromatographic methods (CMs), so group-level differences in CCRS can be small and only visualize trends; statistical support comes from the accompanying Pearson r and 95% CI panels
- The analysis is specific to the RPLC system with eluent pH around 2.7 in the ROASMI models; HILIC generalizability is explicitly future work
- Retention order of compounds with very close chromatographic peaks may be inconsistent across experiments, adding noise to CCRS distributions

## Evidence

- [other] Eluent pH is the main factor affecting the replicability of retention order in the RPLC system.: [evidence span withheld — anchor status: not_found]
- [methods] Generate Fig. 4d-f: violin plots of CCRS grouped by organic-phase modifier, column temperature difference, and other configuration attributes (seaborn violinplot).: [evidence span withheld — anchor status: not_found]
- [intro] The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs).: "The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs)."
