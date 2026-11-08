---
name: microbiome-metabolome-integrative-analysis
description: Use when paired samples have matched taxonomic/functional abundance and
  metabolite abundance matrices (e.g. IBD PRISM, cystic fibrosis, soil datasets) and
  you need to predict the metabolome, identify well-predicted metabolites, and group
  microbes and metabolites with similar interaction patterns.
when_to_use_negative:
- When metabolites are individually modeled from annotation-based references alone
  (e.g. MelonnPan's regularized linear regression), where missing or incorrect annotations
  degrade the model rather than motivating joint multi-metabolite learning.
- When the goal is conditional-probability co-occurrence rather than whole-metabolome
  prediction from taxonomic features (e.g. mmvec-style analysis).
- When samples are unpaired, i.e. microbial and metabolomic profiles do not come from
  the same specimens, since MiMeNet requires paired microbiome-metabolome matrices.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3339
tools:
- name: MiMeNet
  role: 'Primary tool: MLPNN that models the community metabolome from microbial features,
    identifies well-predicted metabolites, and biclusters the interaction score matrix
    into microbe and metabolite modules'
  repo: https://github.com/YDaiLab/MiMeNet
- name: MelonnPan
  role: Baseline linear-regression predictor compared against MiMeNet (Elastic Net
    modeling each metabolite separately)
  repo: https://github.com/biobakery/melonnpan
- name: BiomeNED
  role: Compared neural encoder-decoder predictor of gut metabolites from gut microbiome
  repo: https://github.com/vuongle2/BiomeNED
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009021
    title: 'MiMeNet: Exploring microbiome-metabolome relationships using neural networks'
  source_content_hash: sha256:728a8c8037b85c98ef8525bd4b4373abb497f9e022ea90ffb82be72a95f398b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YDaiLab/MiMeNet
  tools:
  - BiomeNED
  - MelonnPan
  - MiMeNet
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# microbiome-metabolome-integrative-analysis

## Summary

Integrative microbiome-metabolome analysis via MiMeNet: an MLPNN that predicts the community metabolome profile from microbial taxonomic or functional features, then biclusters the interaction-score matrix into co-regulated microbe and metabolite modules.

## When to use

Use when paired samples have matched taxonomic/functional abundance and metabolite abundance matrices (e.g. IBD PRISM, cystic fibrosis, soil datasets) and you need to predict the metabolome, identify well-predicted metabolites, and group microbes and metabolites with similar interaction patterns.

## When NOT to use

- When metabolites are individually modeled from annotation-based references alone (e.g. MelonnPan's regularized linear regression), where missing or incorrect annotations degrade the model rather than motivating joint multi-metabolite learning.
- When the goal is conditional-probability co-occurrence rather than whole-metabolome prediction from taxonomic features (e.g. mmvec-style analysis).
- When samples are unpaired, i.e. microbial and metabolomic profiles do not come from the same specimens, since MiMeNet requires paired microbiome-metabolome matrices.

## Inputs

- Comma delimited matrix of samples by microbial features (micro), e.g. IBD (PRISM) metagenomic taxonomic or functional counts
- Comma delimited matrix of samples by metabolomic features (metab) matched to the same samples
- Optional external microbiome and metabolome matrices (e.g. IBD External test set) for external validation
- Comma delimited annotation file annotating a subset of metabolite features
- Comma delimited labels file for sample labels to associate clusters with (e.g. diagnosis_PRISM.csv)
- JSON file of MLPNN network hyperparameters

## Outputs

- Per-metabolite Spearman correlation coefficients (SCC) from cross-validated evaluation
- List of well-predicted metabolites (SCC above the 95th percentile threshold)
- Microbe and metabolite modules from biclustering of the interaction score matrix
- Module feature values and AUC comparison of MiMeNet modules vs WGCNA modules vs original features (auc_comparison.csv in the output directory)

## How to apply

First run MiMeNet's data preprocessing: transform both the microbial and metabolomic count matrices into relative abundance (RA) or centered log-ratio (CLR) and filter low abundant microbial and metabolite features. Then perform 10 iterations of 10-fold cross-validation (e.g. num_run_cv 10, num_cv 10) training the MLPNN with hyperparameters supplied as a JSON file (net_params), optionally including an external dataset for validation and an annotation file marking a subset of metabolites. Models are evaluated cross-validated to yield a Spearman correlation coefficient (SCC) per metabolite; metabolites whose SCC lies above the 95th percentile threshold are flagged as well-predicted. Finally MiMeNet constructs a microbe-metabolite interaction score matrix that is biclustered into microbe and metabolite modules, with a labels file (e.g. data/IBD/diagnosis_PRISM.csv) used to associate modules with sample classes; downstream comparison can test whether MiMeNet module features retain predictive power versus WGCNA modules or original features (Fig 5C).

## Related tools

- **MiMeNet** (Primary tool: MLPNN that models the community metabolome from microbial features, identifies well-predicted metabolites, and biclusters the interaction score matrix into microbe and metabolite modules) — https://github.com/YDaiLab/MiMeNet
- **MelonnPan** (Baseline linear-regression predictor compared against MiMeNet (Elastic Net modeling each metabolite separately)) — https://github.com/biobakery/melonnpan
- **BiomeNED** (Compared neural encoder-decoder predictor of gut metabolites from gut microbiome) — https://github.com/vuongle2/BiomeNED

## Examples

```
python MiMeNet_train.py -micro data/IBD/microbe_PRISM.csv -metab data/IBD/metabolite_PRISM.csv -external_micro data/IBD_external/microbe.csv -external_metab data/IBD_external/metabolite.csv -labels data/IBD/diagnosis_PRISM.csv -output results/IBD -net_params net_params.json -micro_norm RA -metab_norm RA -threshold 0.95 -num_run_cv 10 -num_cv 10
```

## Evaluation signals

- Mean Spearman correlation should increase relative to linear baselines (e.g. from 0.108 to 0.309, 0.276 to 0.457, and -0.272 to 0.264 across the three paired datasets).
- Number of well-predicted metabolites should increase (198 to 366, 104 to 143, and 4 to 29).
- RA vs CLR transformation sensitivity can be checked via scatterplots of metabolite correlation prediction per dataset (IBD PRISM, cystic fibrosis, soil).
- External validation on the IBD (External) dataset should reproduce metabolite prediction correlation comparable to or better than NED.
- Downstream AUC comparison should show MiMeNet module features retain predictive power similar to original features while WGCNA module features have a much lower mean AUC (Fig 5C).

## Limitations

- All methods including MiMeNet are limited in providing biological plausibility and mechanistic insights.
- Some metabolites have lower prediction correlations because not all metabolites may be associated with microbes.
- The soil dataset showed a higher threshold value, possibly due to longitudinal observations.
- Procedures to statistically associate microbe-metabolite interactions with host disease are not yet implemented; integration of gene-level metagenomics and modeling host metabolome influence on microbial composition remain future directions.

## Evidence

- [methods] MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features.: "MiMeNet will perform a compositional transformation to relative abundance or centered log-ratio and filter low abundant microbial and metabolite features."
- [methods] well-predicted metabolites threshold: "MiMeNet will then take any metabolite with a SCC evaluation value above the 95th percentile to be well-predicted."
- [methods] biclustering interaction score matrix into modules: "This interaction score matrix is biclustered into microbe and metabolite modules, grouping sets of microbes and metabolites with similar interaction patterns."
- [readme] MiMeNet CLI usage from README: "usage: MiMeNet_train.py [-h] -micro MICRO -metab METAB [-external_micro EXTERNAL_MICRO] [-external_metab EXTERNAL_METAB] [-annotation ANNOTATION] [-labels LABELS] -output OUTPUT"
- [results] wgcna vs mimenet auc comparison: [evidence span withheld — anchor status: not_found]
