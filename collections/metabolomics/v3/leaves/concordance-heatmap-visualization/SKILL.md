---
name: concordance-heatmap-visualization
description: Use when concordance_scores.csv (Cohen's kappa and Pearson values with
  p-values and adjusted p-values) is available and reactions must be classified by
  regulatory class and visualized as RPS-vs-RAS and RPS-vs-FFD heatmaps matching Fig
  4.
when_to_use_negative:
- Upstream statistical tests (rasTtest, mannWhitneyUTest, createMetabolicDataset)
  have not been run, so no concordance_scores.csv or means file exists
- The goal is differential-expression or pathway-enrichment visualization rather than
  discriminating flux regulation at the metabolic and/or transcriptional level
- Reactions lack metabolomics coverage for substrates, so the scatterplot subset cannot
  be populated
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: INTEGRATE
  role: pipeline whose concordanceAnalysis step produces the kappa scores and heatmaps
    visualized by this skill
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: runtime for concordanceAnalysis.py, which generates the concordance datasets
    and figures
- name: GLPK
  role: solver used upstream when sampling flux solutions feeding RPS scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:5b34320a1f6f65548a94759f2906ba7bc4f71381d3a59fc1b7c10e7f495c7bc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - GLPK
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# concordance-heatmap-visualization

## Summary

Renders Cohen's-kappa concordance heatmaps of RPS-vs-RAS and RPS-vs-FFD scores from the INTEGRATE pipeline's concordanceAnalysis step, restricting rows to reactions with RPS-vs-FFD concordance above 0.2, to visually separate transcriptional-only, metabolic-only, and combined regulation classes across the 10 breast cell-line pairs.

## When to use

Use when concordance_scores.csv (Cohen's kappa and Pearson values with p-values and adjusted p-values) is available and reactions must be classified by regulatory class and visualized as RPS-vs-RAS and RPS-vs-FFD heatmaps matching Fig 4.

## When NOT to use

- Upstream statistical tests (rasTtest, mannWhitneyUTest, createMetabolicDataset) have not been run, so no concordance_scores.csv or means file exists
- The goal is differential-expression or pathway-enrichment visualization rather than discriminating flux regulation at the metabolic and/or transcriptional level
- Reactions lack metabolomics coverage for substrates, so the scatterplot subset cannot be populated

## Inputs

- concordance_scores.csv with Cohen's kappa and Pearson correlation values, p-values and adjusted p-values for RPS-vs-RAS and RPS-vs-FFD per reaction
- resultsMetabolomicStep 9 t-test output files (prefix 'resultsMetabolomic')
- medie_Met.csv means file from createMetabolicDataset
- ENGRO2_irrev.xml irreversible metabolic model
- list of input cell-line sample names (e.g. ['MCF102A','SKBR3','MCF7','MDAMB231','MDAMB361'])

## Outputs

- RPS-vs-RAS concordance heatmap
- RPS-vs-FFD concordance heatmap
- Scatterplot of RPS-vs-FFD (x) against RPS-vs-RAS (y) for reactions with all substrate abundances quantified
- Reaction classification into metabolic-only, transcriptional+metabolic, transcriptional-only, or other

## How to apply

Run the INTEGRATE concordanceAnalysis script with weight='linear' to compute linear-weighted Cohen's kappa concordance scores for RPS-vs-RAS and RPS-vs-FFD per reaction, using the Step 9 outputs (resultsMetabolomic prefix, medie_Met.csv means file) and the ENGRO2_irrev.xml model. Then filter reactions to those whose RPS-vs-FFD concordance exceeds 0.2 and render two heatmaps: one of RPS-vs-RAS scores and one of RPS-vs-FFD scores across the 10 cell-line pairs; additionally draw a scatterplot of RPS-vs-FFD (x) against RPS-vs-RAS (y) restricted to reactions whose substrates are all quantified in the metabolomics dataset. Classify each reaction as metabolic-only, transcriptional+metabolic, transcriptional-only, or other from its concordance pattern and check the heatmaps against the S4/S5 supplementary concordance tables.

## Related tools

- **INTEGRATE** (pipeline whose concordanceAnalysis step produces the kappa scores and heatmaps visualized by this skill) — https://github.com/qLSLab/integrate
- **Python** (runtime for concordanceAnalysis.py, which generates the concordance datasets and figures)
- **GLPK** (solver used upstream when sampling flux solutions feeding RPS scores)

## Examples

```
python pipeline/concordanceAnalysis.py
```

## Evaluation signals

- concordance_scores.csv contains Cohen's kappa and Pearson values with p-values and adjusted p-values for RPS-vs-RAS and RPS-vs-FFD
- the heatmap includes only reactions whose RPS-vs-FFD concordance exceeds 0.2
- rendered heatmaps match Fig 4 and the S4/S5 supplementary concordance tables
- scatterplot includes only reactions whose substrates all have quantified metabolomics abundances
- each reaction is assigned exactly one of the four classes: metabolic-only, transcriptional+metabolic, transcriptional-only, or other

## Limitations

- Reactions lacking quantified substrate abundances are excluded from the scatterplot
- Concordance is computed per cell-line pair, so results depend on the 10 pairwise comparisons of the five breast cell lines
- The pipeline was tested only with the GLPK solver

## Evidence

- [readme] A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, the pvalues and the adjusted pvalues: [evidence span withheld — anchor status: not_found]
- [intro] The intersection of the two output datasets discriminates fluxes regulated at the metabolic and/or gene expression level.: "The intersection of the two output datasets discriminates fluxes regulated at the metabolic and/or gene expression level."
- [results] After Benjamini-Hochberg FDR correction, the concordance between RPSs and FFDs was statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE: [evidence span withheld — anchor status: not_found]
- [readme] Step 10: Concordance data analysis — Aim: create the results (dataset and figures) for concordance analysis: "Step 10: Concordance data analysis — Aim: create the results (dataset and figures) for concordance analysis"
