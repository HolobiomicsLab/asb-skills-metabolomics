---
name: reaction-activity-score-statistical-testing
description: Use when comparing transcriptomics-derived RAS scores and sampled-flux
  FFD significance across sample pairs to identify reactions whose flux variation
  is concordant with substrate-availability (metabolic) versus enzyme-expression (transcriptional)
  regulation in a constraint-based model such as.
when_to_use_negative:
- When only transcriptomics data are available and no intracellular metabolomics /
  sampled flux solutions exist to compute FFD concordance.
- When the goal is to test metabolomics differences directly (e.g. raw metabolite
  abundance tests) rather than to classify reaction-level flux regulation; use the
  metabolomics t-test outputs alone.
- When the input model is fully reversible with no irreversible-split model available,
  since RAS t-tests are computed on the irreversible-split reaction set.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: rasTtest
  role: Computes pairwise t-tests on RAS scores after splitting RAS over the irreversible
    reactions of the cell-relative model.
  repo: https://github.com/qLSLab/integrate
- name: mannWhitneyUTest
  role: Runs the Mann-Whitney U test on randomly sampled steady-state flux solutions
    to obtain FFD significance per cell-line pair.
  repo: https://github.com/qLSLab/integrate
- name: createMetabolicDataset
  role: Builds the metabolomic statistical-test dataset with quality filter 1 and
    valLog=1.2, producing per-pair t-tests and log2 ratios between cell-line means.
  repo: https://github.com/qLSLab/integrate
- name: concordanceAnalysis
  role: Computes linear-weighted Cohen's kappa concordance of RPS vs RAS and RPS vs
    FFD, plus Pearson correlations, p-values, and adjusted p-values, and renders heatmaps/scatterplot.
  repo: https://github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:4155b79854e7353833b11a3597523f63034c92b87e3d0a0a763c098b38d357de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - INTEGRATE
  - Python
  - concordanceAnalysis
  - createMetabolicDataset
  - mannWhitneyUTest
  - rasTtest
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# reaction-activity-score-statistical-testing

## Summary

Statistical testing of Reaction Activity Scores (RAS) and flux fold-differences (FFD) from INTEGRATE: pairwise t-tests on RAS across cell-line pairs, Mann-Whitney U tests on randomly sampled steady-state flux solutions, and concordance analysis with Cohen's kappa to classify flux regulation as metabolic, transcriptional, or combined.

## When to use

Use when comparing transcriptomics-derived RAS scores and sampled-flux FFD significance across sample pairs to identify reactions whose flux variation is concordant with substrate-availability (metabolic) versus enzyme-expression (transcriptional) regulation in a constraint-based model such as ENGRO2.

## When NOT to use

- When only transcriptomics data are available and no intracellular metabolomics / sampled flux solutions exist to compute FFD concordance.
- When the goal is to test metabolomics differences directly (e.g. raw metabolite abundance tests) rather than to classify reaction-level flux regulation; use the metabolomics t-test outputs alone.
- When the input model is fully reversible with no irreversible-split model available, since RAS t-tests are computed on the irreversible-split reaction set.

## Inputs

- Normalized RAS scores per cell line (e.g. ENGRO2_wNormalizedRAS.csv output of getNormalizedRAS)
- Irreversible cell-relative SBML/mat models from rasIntegration and MATLAB-based splitting (e.g. ENGRO2_MCF102A_wIrrRxns)
- Randomly sampled flux solutions from each cell relative model (randomSampling_nSol_<nSamples>_<cellLine>_<timeStamp>.csv)
- Metabolomics dataset (metabolomics_LM.csv) with metabolite-ID conversion file metsEngroVsMetabolomics.csv
- Per-pair t-test output files of createMetabolicDataset and prefix 'resultsMetabolomic'
- Cell-line list, biomass reaction name, and per-pair pair labels (c_1 vs c_2)

## Outputs

- ras_<c1>_vs_<c2>.csv pairwise t-test files for each cell-line pair
- Per-pair Mann-Whitney U test files for flux fold-differences (FFD)
- medie_Met.csv means file and log2-ratio / t-test outputs from createMetabolicDataset
- concordance_analysis dataset of Cohen's kappa (RPS vs RAS, RPS vs FFD), Pearson correlations, p-values, and adjusted p-values per reaction
- Heatmaps of RPS-vs-RAS and RPS-vs-FFD (restricted to reactions with RPS-vs-FFD concordance > 0.2) and a scatterplot of RPS-vs-FFD (x) vs RPS-vs-RAS (y)
- Reaction classification into metabolic-only, transcriptional+metabolic, transcriptional-only, or other

## How to apply

Split RAS scores over the irreversible reactions of the cell-relative model (e.g. ENGRO2_MCF102A_wIrrRxns) and run pairwise t-tests on RAS for each of the 10 cell-line pairs, producing 'ras_<c1>_vs_<c2>.csv' files. Separately, run Mann-Whitney U tests on the nSamples randomly sampled steady-state flux solutions from each cell relative model to obtain flux fold-difference (FFD) significance. Build the metabolomic statistical-test dataset with quality filter 1 and valLog=1.2 (log-ratio threshold) via createMetabolicDataset, computing per-pair t-tests and log2 ratios between cell-line means. Finally, run concordanceAnalysis with linear-weighted Cohen's kappa to compute RPS-vs-RAS and RPS-vs-FFD concordance scores, Pearson correlations, p-values, and adjusted p-values per reaction; select reactions with RPS-vs-FFD concordance > 0.2 and render RPS-vs-RAS and RPS-vs-FFD heatmaps, plus a scatterplot of RPS-vs-FFD (x) against RPS-vs-RAS (y) for reactions whose all substrate abundances are quantified, classifying each reaction as metabolic-only, transcriptional+metabolic, transcriptional-only, or other.

## Related tools

- **rasTtest** (Computes pairwise t-tests on RAS scores after splitting RAS over the irreversible reactions of the cell-relative model.) — https://github.com/qLSLab/integrate
- **mannWhitneyUTest** (Runs the Mann-Whitney U test on randomly sampled steady-state flux solutions to obtain FFD significance per cell-line pair.) — https://github.com/qLSLab/integrate
- **createMetabolicDataset** (Builds the metabolomic statistical-test dataset with quality filter 1 and valLog=1.2, producing per-pair t-tests and log2 ratios between cell-line means.) — https://github.com/qLSLab/integrate
- **concordanceAnalysis** (Computes linear-weighted Cohen's kappa concordance of RPS vs RAS and RPS vs FFD, plus Pearson correlations, p-values, and adjusted p-values, and renders heatmaps/scatterplot.) — https://github.com/qLSLab/integrate

## Examples

```
python pipeline/rasTtest.py && python pipeline/mannWhitneyUTest.py <timeStampInput> <nSamples> && python pipeline/createMetabolicDataset.py && python pipeline/concordanceAnalysis.py
```

## Evaluation signals

- The concordance_scores.csv contains Cohen's kappa and Pearson values with p-values and adjusted p-values for both RPS-vs-RAS and RPS-vs-FFD per reaction.
- The heatmap includes only reactions whose RPS-vs-FFD concordance exceeds 0.2.
- The scatterplot of RPS-vs-FFD (x) against RPS-vs-RAS (y) is restricted to reactions with all substrate abundances quantified.
- Per-pair output files exist for each of the 10 cell-line pairs (e.g. ras_<c1>_vs_<c2>.csv) and follow the expected naming convention.
- Reaction classifications (metabolic-only, transcriptional+metabolic, transcriptional-only, other) are consistent with the S4/S5 supplementary concordance tables.

## Limitations

- RAS t-tests are computed on the irreversible-split reaction set, so reversible reactions in the original model must be split beforehand.
- The FFD significance relies on random sampling of the feasible flux region; results depend on nSamples and nBatches choices.
- The metabolomic test dataset uses valLog=1.2 as a fixed threshold above which the ratio between means of two cell lines is considered statistically different; changing this threshold alters downstream concordance results.

## Evidence

- [supplementary] After Benjamini-Hochberg FDR correction, the concordance between RPSs and FFDs was statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE: [evidence span withheld — anchor status: not_found]
- [readme] Step 8: RAS t-test ... Usage: `python pipeline/rasTtest.py`: "Step 8: RAS t-test ... Aim: compute the t-test of RAS scores ... Usage: `python pipeline/rasTtest.py`"
- [readme] Step 7: mannWhitneyUTest ... Aim: compute the Mann-Whitney U test: "Step 7: mannWhitneyUTest * Aim: compute the Mann-Whitney U test * Usage: `python pipeline/mannWhitneyUTest.py timeStampInput nSamples`"
- [readme] valLog: value above which the ratio between the means of two cell lines are considered statistically different. Default value: 1.2: "valLog: value above which the ratio between the means of two cell lines are considered statistically different. Default value: 1.2"
- [readme] weight: weight on the Kappa Cohen. Default value: 'linear': "weight: weight on the Kappa Cohen. Default value: 'linear'"
