---
name: cohen-kappa-concordance-scoring
description: Use when flux-sampling FFD classifications and transcriptomics-derived
  RAS scores must be compared against metabolomics-derived RPS signs for the same
  irreversible reactions, to score per-reaction concordance across cell-line pairs.
when_to_use_negative:
- When the user only needs single-omics flux prediction (e.g. pure RAS scoring or
  pure FFD sampling) without cross-omics concordance — kappa scoring requires all
  three classification layers (RPS, RAS, FFD) computed on the same irreversible reaction
  set.
- When the model's reactions lack paired metabolomics and transcriptomics measurements
  per cell line; kappa on incomplete or non-aligned reaction-pair matrices will not
  reproduce the paper's regulatory categories.
- When the goal is pathway enrichment or gene-metabolite correlation analysis instead
  of per-reaction regulation-category assignment — the article explicitly notes those
  alternative framings do not capture multi-level regulation.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3174
tools:
- name: INTEGRATE
  role: Provides the concordanceAnalysis script that computes linearly-weighted Cohen's
    kappa and Pearson correlation per reaction across cell-line pairs
  repo: github.com/QLSLab/integrate
- name: Python
  role: Runtime for concordanceAnalysis and for bootstrap-based empirical p-value
    / Benjamini-Hochberg FDR adjustment on the kappa scores
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:73daf72e93480b746ddcf2bb105e381a70beca2ac0a2163d54ec8e5f3c3096c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GLPK
  - INTEGRATE
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cohen-kappa-concordance-scoring

## Summary

Computes linearly-weighted Cohen's kappa concordance between RPS (metabolomics-derived) and RAS/FFD (transcriptomics- and sampling-derived) reaction classifications in INTEGRATE, scoring per-reaction pairwise variation signs across cell-line pairs and flagging multi-level metabolic regulation.

## When to use

Use when flux-sampling FFD classifications and transcriptomics-derived RAS scores must be compared against metabolomics-derived RPS signs for the same irreversible reactions, to score per-reaction concordance across cell-line pairs.

## When NOT to use

- When the user only needs single-omics flux prediction (e.g. pure RAS scoring or pure FFD sampling) without cross-omics concordance — kappa scoring requires all three classification layers (RPS, RAS, FFD) computed on the same irreversible reaction set.
- When the model's reactions lack paired metabolomics and transcriptomics measurements per cell line; kappa on incomplete or non-aligned reaction-pair matrices will not reproduce the paper's regulatory categories.
- When the goal is pathway enrichment or gene-metabolite correlation analysis instead of per-reaction regulation-category assignment — the article explicitly notes those alternative framings do not capture multi-level regulation.

## Inputs

- resultsMetabolomic files (RPS t-test outputs and log2 mean-ratio table medie_Met.csv from Step 9)
- ras_<c1>_vs_<c2>.csv pairwise RAS t-test outputs from Step 8 (rasTtest)
- Mann-Whitney U test FFD outputs per cell-line pair from Step 7
- ENGRO2_irrev.xml irreversible metabolic model
- Cell-line list e.g. ['MCF102A', 'SKBR3', 'MCF7', 'MDAMB231', 'MDAMB361']

## Outputs

- Dataset of concordance scores (Cohen's kappa and Pearson correlation) for RPS-vs-RAS and RPS-vs-FFD, with p-values and adjusted p-values (e.g. concordance_scores.csv)
- adjusted_pvalues.csv of empirical p-values and BH-adjusted p-values at 5% FDR
- Heatmap of RPS-vs-RAS and RPS-vs-FFD concordance (reactions with RPS-vs-FFD kappa > 0.2)
- Quadrant scatterplot (RPS-vs-FFD on x, RPS-vs-RAS on y) for reactions with all substrate abundances quantified
- Regulatory class assignment per reaction (metabolic-only, transcriptional+metabolic, transcriptional-only, other)

## How to apply

Run concordanceAnalysis with weight='linear' on the three classification tables produced upstream: RPS signs from createMetabolicDataset (data_quality_filter=1, valLog=1.2, log2 mean ratios), RAS t-test outcomes from rasTtest on the split irreversible ENGRO2 model, and FFD Mann-Whitney U outcomes from randomSampling batches (epsilon=1e-4 biomass lower bound). For each reaction and each of the 10 cell-line pairs, compute linearly-weighted Cohen's kappa between RPS-vs-FFD and RPS-vs-RAS variation signs, plus Pearson correlation, and record p-values and Benjamini-Hochberg-adjusted p-values. Reactions with RPS-vs-FFD concordance greater than 0.2 are retained for the heatmap; the adjusted p-value below 5% FDR identifies statistically significant concordance (ACONT, ASPTA forward, RPI, RPE in the ENGRO2 run). A reaction whose kappa sign pattern matches metabolic-only, transcriptional+metabolic, transcriptional-only, or other categories is then assigned to the corresponding regulatory class.

## Related tools

- **INTEGRATE** (Provides the concordanceAnalysis script that computes linearly-weighted Cohen's kappa and Pearson correlation per reaction across cell-line pairs) — github.com/QLSLab/integrate
- **Python** (Runtime for concordanceAnalysis and for bootstrap-based empirical p-value / Benjamini-Hochberg FDR adjustment on the kappa scores)

## Examples

```
python pipeline/concordanceAnalysis.py
```

## Evaluation signals

- After BH FDR correction at 5%, the reactions ACONT, ASPTA (forward), RPI, and RPE must surface as significantly concordant — use this as the reproduction check.
- Only reactions with RPS-vs-FFD Cohen's kappa > 0.2 should appear in the concordance heatmap; reactions below this threshold must be excluded.
- The output table must contain, for each reaction, both RPS-vs-RAS and RPS-vs-FFD kappa values, Pearson correlations, raw p-values, and adjusted p-values.
- Each reaction must be assignable to exactly one of the four regulatory categories (metabolic-only, transcriptional+metabolic, transcriptional-only, other) based on its kappa sign pattern.
- Re-running the pipeline on Recon3D restricted to ENGRO2-active exchange reactions should distribute reactions across the same regulatory categories and recover RPE as transcriptionally and metabolically regulated in both models.

## Limitations

- The kappa statistic is a linearly-weighted variant tuned to sign-concordance of pairwise variation across exactly two conditions per cell-line pair; concordance on more than two conditions is not supported in this skill's scope.
- Concordance scores are sensitive to the upstream thresholds (valLog=1.2 for metabolite mean ratios, epsilon=1e-4 for biomass lower bound in sampling); changing these alters kappa magnitudes and category assignments.
- Reactions with missing substrate abundance quantification are excluded from the quadrant scatterplot, so concordance interpretation for partially measured reactions is incomplete.
- The empirically significant concordant set (ACONT, ASPTA forward, RPI, RPE) is specific to the ENGRO2 model and the breast cell-line panel; it is not a universal signature.

## Evidence

- [methods] Run concordanceAnalysis with linear-weighted Cohen's kappa to compute RPS-vs-RAS and RPS-vs-FFD concordance scores: [evidence span withheld — anchor status: not_found]
- [results] After FDR correction of the p-values, the concordance between RPSs and FFDs resulted statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE: "After FDR correction of the p-values, the concordance between RPSs and FFDs resulted statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE."
- [readme] A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues: "A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues"
- [readme] Usage: `python pipeline/concordanceAnalysis.py`: "Usage: `python pipeline/concordanceAnalysis.py`"
- [readme] valLog: value above which the ratio between the means of two cell lines are considered statistically different. Default value: 1.2: "valLog: value above which the ratio between the means of two cell lines are considered statistically different. Default value: 1.2"
- [results] After Benjamini-Hochberg FDR correction, the concordance between RPSs and FFDs was statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE: [evidence span withheld — anchor status: not_found]
- [supplementary] Classify each reaction as metabolic-only, transcriptional+metabolic, transcriptional-only, or other from its concordance pattern: [evidence span withheld — anchor status: not_found]
