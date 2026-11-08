---
name: cross-dataset-retention-order-alignment
description: Use when you have paired RPLC retention datasets (e.
when_to_use_negative:
- Data come from HILIC rather than RPLC systems — the ROASMI/CCRS replicability analysis
  applies only to RPLC, and HILIC generalizability is explicitly left as future work
- You need absolute RT values rather than relative elution order — CCRS measures correlation
  of retention sequences, which is robust to RT offsets but discards magnitude information
- Datasets share too few compounds to form a meaningful retention-sequence correlation
  per pair
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: ROASMI
  role: Source framework whose analysis pipeline (data folder, figure scripts) contains
    the 59 paired RPLC datasets and the CCRS vs ΔpH/ΔF analysis code
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Molecular embedding module (D-MPNN) underlying ROASMI's retention prediction;
    relevant when the aligned retention orders are later used to train/extend retention-order
    models
  repo: https://github.com/chemprop/chemprop
- name: msms_rt_score_integration
  role: Provides the probabilistic MS/MS + retention-time integration context for
    which retention-order replicability (CCRS) matters
  repo: https://github.com/aalto-ics-kepaco/msms_rt_score_integration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:12fded837e6405e3e0d3501d3b28eeda731fdad1c25fb5e431d58dc70ca24471
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  - msms_rt_score_integration
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# cross-dataset-retention-order-alignment

## Summary

Compute the CCRS (correlation coefficient of retention sequences) between paired RPLC retention orders of shared compounds across dataset pairs, then regress/stratify CCRS against chromatographic differences (ΔpH, ΔF, temperature) to identify which chromatographic factor drives retention-order replicability. This skill operationalizes the finding that eluent pH is the main factor affecting replicability of retention order in RPLC.

## When to use

Use when you have paired RPLC retention datasets (e.g., RepoRT, PredRet, Metabolome Workbench, MetaboLights) sharing compounds, and you need to quantify how consistently compound elution order replicates and which chromatographic factor (eluent pH, organic-phase %, temperature) explains the agreement.

## When NOT to use

- Data come from HILIC rather than RPLC systems — the ROASMI/CCRS replicability analysis applies only to RPLC, and HILIC generalizability is explicitly left as future work
- You need absolute RT values rather than relative elution order — CCRS measures correlation of retention sequences, which is robust to RT offsets but discards magnitude information
- Datasets share too few compounds to form a meaningful retention-sequence correlation per pair

## Inputs

- 59 paired RPLC retention datasets (RepoRT, PredRet, Metabolome Workbench, MetaboLights sources) collated in the paper's data folder
- Per-dataset chromatographic configuration attributes: eluent pH, organic-phase percentage/modifier, column temperature
- List of shared compounds between each dataset pair with their retention orders

## Outputs

- Per-pair CCRS values and pair attributes (results/ccrs_per_pair.csv)
- Composite figure fig4_a_to_h.png: ΔpH vs CCRS and ΔF vs CCRS scatter plots with Pearson r and 95% CI, ΔpH≤1 subset summary, and violin plots of CCRS grouped by modifier, temperature difference and configuration attributes

## How to apply

Load the 59 paired RPLC retention datasets collated from RepoRT, PredRet, Metabolome Workbench and MetaboLights sources in the paper's data folder. For each dataset pair, compute the CCRS between paired retention sequences of the shared compounds. For each pair, compute ΔpH, ΔF (organic-phase percentage difference) and column temperature difference, then scatter-plot ΔpH vs CCRS with Pearson r and 95% CI (scipy.stats.pearsonr, matplotlib); summarize the ΔpH≤1 subset reporting mean CCRS and its 95% CI. Contrast pH against other factors with violin plots of CCRS grouped by organic-phase modifier, temperature difference and other configuration attributes (seaborn violinplot), plus ΔF vs CCRS scatter plots, to justify the conclusion that eluent pH is the dominant driver of retention-order replicability.

## Related tools

- **ROASMI** (Source framework whose analysis pipeline (data folder, figure scripts) contains the 59 paired RPLC datasets and the CCRS vs ΔpH/ΔF analysis code) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Molecular embedding module (D-MPNN) underlying ROASMI's retention prediction; relevant when the aligned retention orders are later used to train/extend retention-order models) — https://github.com/chemprop/chemprop
- **msms_rt_score_integration** (Provides the probabilistic MS/MS + retention-time integration context for which retention-order replicability (CCRS) matters) — https://github.com/aalto-ics-kepaco/msms_rt_score_integration

## Examples

```
git clone https://github.com/FangYuan717/ROASMI.git; conda env create -f environment.yml; conda activate ROASMI; python code/analysis_ccrs.py  # reproduce CCRS vs ΔpH/ΔF analysis from data/ datasets
```

## Evaluation signals

- The ΔpH vs CCRS scatter yields a positive Pearson correlation with a 95% confidence interval excluding zero
- The ΔpH≤1 subset shows a higher mean CCRS (with 95% CI) than the overall pool of pairs, consistent with pH being the main driver
- Violin plots of CCRS grouped by organic-phase modifier and column temperature difference show weaker separation than pH-based grouping
- results/ccrs_per_pair.csv contains one row per dataset pair with both CCRS and the chromatographic attributes (ΔpH, ΔF, temperature) populated
- The composite figure exports as figures/fig4_a_to_h.png with panels 4a–4h matching the described layout

## Limitations

- When chromatographic peaks of two compounds are close enough together, their retention order may not be consistent across experiments, adding noise to CCRS
- RT outliers caused by slight changes in experimental conditions are unresolved and require further mechanistic study
- The underlying ROASMI model applies only to RPLC systems (eluent pH around 2.7 for the shipped models) and to conventional compounds within elemental restrictions of C, H, N, O, P, and S

## Evidence

- [results] Eluent pH is the main factor affecting the replicability of retention order in the RPLC system.: [evidence span withheld — anchor status: not_found]
- [intro] The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs).: "The underutilization of retention data in automated annotation tools is due primarily to the poor replicability of RTs across different chromatographic methods (CMs)."
- [discussion] when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments: "when the chromatographic peaks of two compounds are close enough together, their order of retention may not be consistent across experiments."
- [readme] recreate the entire analysis for the manuscript: "this repository also contains data that allows researchers to recreate the entire analysis for the manuscript"
- [discussion] ROASMI currently applies only to RPLC systems: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
