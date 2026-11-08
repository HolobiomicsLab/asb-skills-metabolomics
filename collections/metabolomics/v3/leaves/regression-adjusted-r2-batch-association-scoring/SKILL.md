---
name: regression-adjusted-r2-batch-association-scoring
description: Use when a log2-scaled metabolomics matrix (samples as rows, batch labels
  in the first column) has been corrected by multiple batch-effect models and you
  need a quantitative, comparable metric of residual metabolite-vs-batch association
  to select the model.
when_to_use_negative:
- Datasets with more than ~2000 features (variables) — dbnormSCORE is suggested for
  less than 2000 features for better computational speed
- Input data are not normalized/log2-scaled — the README requires data to be scaled
  on log2 before upload
- The goal is exploratory visualization (PCA, PDF/violin plots per metabolite) rather
  than quantitative model comparison — use Visodbnorm or ProfPlot* functions instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: dbnorm
  role: R package providing dbnormSCORE and the batch-correction functions whose outputs
    are scored
  repo: https://github.com/NBDZ/dbnorm
- name: ber
  role: Batch-effect removal model (linear fitting of location/scale parameters, originally
    for microarray data) whose corrected output is scored by dbnormSCORE
- name: ComBat
  role: Empirical Bayes location/scale batch-correction model (sva package) applied
    in parametric and nonparametric settings via dbnormPcom/dbnormNPcom before scoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:223d69bfc87b3dc6d113456a53391ccf6cd1fe50dc7f736874c7856681813aa2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - ComBat
  - R
  - ber
  - dbnorm
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# regression-adjusted-r2-batch-association-scoring

## Summary

Scores batch-effect correction performance by regressing each metabolite's intensity on batch level and taking the maximum adjusted-R² (dbnormSCORE, 0–1 scale), enabling direct comparison of lowess, ber, bagging ber, and parametric/nonparametric ComBat on log2-transformed metabolomics matrices.

## When to use

Use when a log2-scaled metabolomics matrix (samples as rows, batch labels in the first column) has been corrected by multiple batch-effect models and you need a quantitative, comparable metric of residual metabolite-vs-batch association to select the best model.

## When NOT to use

- Datasets with more than ~2000 features (variables) — dbnormSCORE is suggested for less than 2000 features for better computational speed
- Input data are not normalized/log2-scaled — the README requires data to be scaled on log2 before upload
- The goal is exploratory visualization (PCA, PDF/violin plots per metabolite) rather than quantitative model comparison — use Visodbnorm or ProfPlot* functions instead

## Inputs

- Log2-transformed metabolomics intensity matrix in .csv format (samples in rows, metabolite features in columns, batch levels in the first column), e.g. SKIPOGH targeted LC-MS data with 1079 samples, 239 metabolites, 11 batches
- Batch-corrected versions of that matrix produced by dbnormBer, dbnormBagging, dbnormPcom, or dbnormNPcom

## Outputs

- Score table of maximum adjusted-R² per correction model on the 0–1 adj-R² scale
- Correlation plot and score plot compiled into a PDF saved in the working directory
- .csv files of adjusted-R² per metabolite per model, saved to a temporary folder initiated with Rtmpe

## How to apply

Load the log2-transformed matrix in .csv format (rows = independent experiments, columns = features, batch levels in the first column) and compute, per metabolite, the regression of intensity on batch level, extracting the adjusted coefficient of determination (adj-R²) for the raw data. Correct the data with the candidate dbnorm models — dbnormBer and dbnormBagging (ber and ber-bagging with partial bagging, n=150 bootstrap samples), and dbnormPcom/dbnormNPcom (ComBat empirical Bayes location/scale estimation from the sva package) — then recompute the metabolite-vs-batch adj-R² on each corrected dataset. Run dbnormSCORE, which calculates the score from the maximum adj-R² (0 = no batch association, 1 = fully batch-explained) and emits correlation and score plots compiled into a PDF plus .csv outputs. Compare the score table across raw and corrected datasets: lower maximum adj-R² indicates better batch-drift removal. The rationale is that a model whose corrected data no longer associates metabolite intensities with batch (maximum adj-R² near 0) has accommodated the technical variability, whereas residual high adj-R² (e.g. lowess 0.78, nonparametric ComBat 0.60 in SKIPOGH) reveals incomplete correction.

## Related tools

- **dbnorm** (R package providing dbnormSCORE and the batch-correction functions whose outputs are scored) — https://github.com/NBDZ/dbnorm
- **ber** (Batch-effect removal model (linear fitting of location/scale parameters, originally for microarray data) whose corrected output is scored by dbnormSCORE)
- **ComBat** (Empirical Bayes location/scale batch-correction model (sva package) applied in parametric and nonparametric settings via dbnormPcom/dbnormNPcom before scoring)

## Examples

```
data <- read.csv("path/to/mydata.csv", sep=",", header=TRUE, row.names=1); library(dbnorm); dbnormSCORE(data)
```

## Evaluation signals

- Score table exists and each model score is bounded to the 0–1 adj-R² scale
- Reproduces SKIPOGH Figure 1B–C ranges: raw and lowess up to 0.78 maximum residual variability, nonparametric ComBat up to 0.60, ber and parametric ComBat below 0.01
- Raw-data scores confirm >50% of variability was batch-explained for most metabolites in SKIPOGH
- PDF outputs contain both the correlation plot and the score plot; .csv files are written to the Rtmpe temporary folder
- Parametric ComBat, ber, and bagging ber corrected data show no residual metabolite-to-batch association, while lowess and nonparametric ComBat retain some

## Limitations

- dbnormSCORE is suggested for less than 2000 features (variables) for better computational speed
- Bagging model is performed using partial bagging with n=150 bootstrap samples; other bootstrap sizes are not covered by this workflow
- Software version for dbnorm and precise versions of ComBat and ber are not specified in the article; exact scores are dataset-dependent (values reported here are for SKIPOGH only)

## Evidence

- [methods] dbnormSCORE: [evidence span withheld — anchor status: not_found]
- [results] ber/parametric ComBat performance: "the datasets corrected by ber and parametric Com Bat functions presented a similar performance in this study, with a maximum variability associated with the batch < 0.01 (1%)"
- [results] lowess residual: "the maximum variability detected for a metabolite was 0.78 (78%) for the lowess-corrected dataset (Supplementary Table 2)"
- [readme] README scoring description: "the performance of applied models are presented by a score calculated based on the maximum variability explained by the batch level"
- [readme] README output formats: "Graphical check such as *Correlation* plot and *Score* plot compiled into a **PDF** file (saved in the working directory) and **.csv** files"
