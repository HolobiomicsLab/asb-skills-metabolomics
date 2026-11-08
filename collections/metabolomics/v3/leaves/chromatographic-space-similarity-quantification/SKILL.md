---
name: chromatographic-space-similarity-quantification
description: Use when you have paired RPLC retention datasets (e.g., RepoRT, PredRet,
  Metabolome Workbench, MetaboLights sources) sharing compounds across different chromatographic
  methods and need to measure retention-order replicability or attribute it to specific
  method differences.
when_to_use_negative:
- Datasets are HILIC rather than RPLC — the paper's retention-order replicability
  analysis (and ROASMI) applies only to the RPLC system
- The two datasets being compared share too few compounds to form a meaningful paired
  retention sequence
- You need absolute RT prediction rather than retention-order replicability quantification
tools:
- name: ROASMI
  role: Retention-order model whose generalizability motivates and extends the ChrSS/CCRS
    replicability analysis
  repo: https://github.com/FangYuan717/ROASMI
- name: chemprop
  role: Molecular embedding module (D-MPNN extension) underlying retention-order prediction
    evaluated on paired RPLC datasets
  repo: https://github.com/chemprop/chemprop
- name: PyTorch
  role: Deep-learning backend required to run the ROASMI/chemprop models in the reproducibility
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-025-00968-8
    title: 'ROASMI: accelerating small molecule identification by repurposing retention
      data'
  source_content_hash: sha256:01d2a6fe3c18cb74b62b0282ef0e27c7e5557a60a4c5d0a454d997d94341791c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FangYuan717/ROASMI
  tools:
  - PyTorch
  - ROASMI
  - chemprop
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chromatographic-space-similarity-quantification

## Summary

Quantifies how similar two chromatographic configurations are (ChrSS) by correlating paired retention sequences via CCRS (correlation coefficient of retention sequences) and relating it to ΔpH, ΔF, and Δ column temperature, to identify which factor drives retention-order replicability in RPLC.

## When to use

Use when you have paired RPLC retention datasets (e.g., RepoRT, PredRet, Metabolome Workbench, MetaboLights sources) sharing compounds across different chromatographic methods and need to measure retention-order replicability or attribute it to specific method differences.

## When NOT to use

- Datasets are HILIC rather than RPLC — the paper's retention-order replicability analysis (and ROASMI) applies only to the RPLC system
- The two datasets being compared share too few compounds to form a meaningful paired retention sequence
- You need absolute RT prediction rather than retention-order replicability quantification

## Inputs

- 59 paired RPLC retention datasets (RepoRT, PredRet, Metabolome Workbench, MetaboLights sources)
- Per-dataset chromatographic configuration metadata (eluent pH, organic-phase modifier and percentage, column temperature)
- Retention order tables of shared compounds per dataset pair

## Outputs

- results/ccrs_per_pair.csv with per-pair CCRS values and pair attributes (ΔpH, ΔF, ΔT, configuration attributes)
- Composite figure figures/fig4_a_to_h.png (scatter plots with Pearson r and 95% CI, ΔpH≤1 subset summary, violin plots by configuration attributes)

## How to apply

Load the paired retention datasets and, for each dataset pair, compute the CCRS between the paired retention orders of the shared compounds. For each pair, compute ΔpH, ΔF (organic-phase percentage difference), and Δ column temperature, and record the chromatographic configuration attributes. Test associations by regressing or plotting ΔpH vs CCRS with Pearson r and its 95% confidence interval (scipy.stats.pearsonr), and use seaborn violinplots of CCRS grouped by organic-phase modifier, temperature difference, and other configuration attributes to isolate factor effects. Restrict a sensitivity analysis to the ΔpH≤1 subset and report the mean CCRS with its 95% CI; contrast with ΔF vs CCRS scatter plots. The rationale is that RT replicability across methods is poor, and the article establishes that eluent pH is the main factor affecting replicability of retention order in the RPLC system.

## Related tools

- **ROASMI** (Retention-order model whose generalizability motivates and extends the ChrSS/CCRS replicability analysis) — https://github.com/FangYuan717/ROASMI
- **chemprop** (Molecular embedding module (D-MPNN extension) underlying retention-order prediction evaluated on paired RPLC datasets) — https://github.com/chemprop/chemprop
- **PyTorch** (Deep-learning backend required to run the ROASMI/chemprop models in the reproducibility pipeline)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['delta_pH'], df['ccrs']); df.to_csv('results/ccrs_per_pair.csv')
```

## Evaluation signals

- Every dataset pair with shared compounds yields a single CCRS value in [-1, 1]; results/ccrs_per_pair.csv must contain one row per pair plus ΔpH, ΔF, and Δ temperature columns
- The ΔpH vs CCRS scatter must report a Pearson r with a 95% confidence interval computed via scipy.stats.pearsonr
- The ΔpH≤1 subset summary must report mean CCRS with a 95% CI, and this subset should show higher replicability than the full set
- Violin plots grouping CCRS by organic-phase modifier, temperature difference, and other configuration attributes should reproduce the paper's finding that pH is the dominant factor, with ΔF vs CCRS showing a weaker relationship
- Downstream sanity check: pairs with very close chromatographic peaks should show inconsistent CCRS, reflecting the paper's caveat about near-co-eluting compounds

## Limitations

- Retention order of very close chromatographic peaks may not be consistent across experiments, limiting CCRS reliability for near-co-eluting pairs
- RT outliers resulting from slight changes in experimental conditions are unresolved and require further mechanistic study
- The analysis and the associated ROASMI model apply only to the RPLC system; HILIC retention replicability is future work

## Evidence

- [intro] Open question on required ChrSS: "Nonetheless, determining the degree of ChrSS required for replicable retention orders remains to be explored."
- [results] Main finding on pH as driver: [evidence span withheld — anchor status: not_found]
- [methods] CCRS computation from paired retention orders: [evidence span withheld — anchor status: not_found]
- [intro] RT variability across configurations as motivation (ST001683): "The distribution of differences in experimental RTs observed between different configurations (source data: ST001683 dataset"
- [discussion] RPLC-only scope: "ROASMI currently applies only to RPLC systems, which limits its application to a specific chromatographic space."
- [readme] Repository provenance for the analysis data: "this repository also contains data that allows researchers to recreate the entire analysis for the manuscript"
