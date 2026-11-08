---
name: log2fold-change-thresholding
description: Use when you have a batch-corrected LC-MS feature table from a two-group
  dietary/condition contrast and need a statistically defensible list of differentially
  abundant metabolites with a volcano plot.
when_to_use_negative:
- Input is a raw (uncorrected) LC-MS feature table that still carries batch/drift
  effects — apply dbnorm batch correction first; raw analysis under-detects HFD-altered
  metabolites compared to ber-corrected data.
- You need an exhaustive, unbiased list of all changed metabolites without a fold-change
  stringency — the Log2FC > 1.5 cutoff is deliberately stringent and filters out moderate
  effect sizes.
- Dataset has > 2000 features and you want dbnorm's model selection — this skill is
  the downstream differential step, not the correction step.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: dbnorm
  role: Upstream batch effect correction producing the ber-corrected feature table
    on which Log2FC/adj-p differential analysis is performed; also generates the volcano/differential
    outputs downstream
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Statistical environment in which the linear regression per feature, BH correction,
    and volcano plotting are executed
- name: ggplot2
  role: R plotting library used to draw the volcano plot of Log2FC vs -log10(adjusted
    p-value)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:d56fe3786df3569d716e39a0436a4dcdb2b286653f6e52d99e2b58c245185132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - dbnorm
  - ggplot2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# log2fold-change-thresholding

## Summary

Differential analysis of untargeted LC-MS metabolomics features in which metabolites altered by a dietary intervention (e.g. 8-week HFD vs control in mouse subcutaneous adipose tissue) are selected via Log2FC > 1.5 and BH-adjusted p < 0.05, then visualized as a volcano plot.

## When to use

Use when you have a batch-corrected LC-MS feature table from a two-group dietary/condition contrast and need a statistically defensible list of differentially abundant metabolites with a volcano plot.

## When NOT to use

- Input is a raw (uncorrected) LC-MS feature table that still carries batch/drift effects — apply dbnorm batch correction first; raw analysis under-detects HFD-altered metabolites compared to ber-corrected data.
- You need an exhaustive, unbiased list of all changed metabolites without a fold-change stringency — the Log2FC > 1.5 cutoff is deliberately stringent and filters out moderate effect sizes.
- Dataset has > 2000 features and you want dbnorm's model selection — this skill is the downstream differential step, not the correction step.

## Inputs

- Batch-corrected log2-scaled LC-MS feature table (.csv) for the 8-week HFD vs control mouse subcutaneous adipose tissue comparison (rows = independent experiments, columns = features/variables)
- Diet/condition metadata (batch levels and treatment group per sample)

## Outputs

- CSV list of significant metabolites/features (Log2FC > 1.5, BH-adjusted p < 0.05)
- Volcano plot (PNG) of Log2FC vs -log10(adjusted p-value) with significant features highlighted

## How to apply

Load the log2-scaled feature matrix (rows = samples, columns = features) from the batch-corrected dataset, e.g. the ber-corrected output of dbnorm, and fit a linear regression model per metabolic feature to estimate the fold change and p-value of the diet effect (HFD vs control). Adjust p-values with Benjamini-Hochberg correction, then select features satisfying Log2FC > 1.5 AND adjusted p-value < 0.05 — the article's stated significance criteria for drawing a list of metabolites with differential level. Generate a volcano plot of Log2FC against -log10(adjusted p) for all features, highlighting the significant subset (R + ggplot2). Export the significant feature list to CSV. Apply this after batch effect correction, not on raw data: ber-corrected data yielded a more exhaustive candidate list for HFD effects, with several metabolites (mostly lipids) significant only after correction.

## Related tools

- **dbnorm** (Upstream batch effect correction producing the ber-corrected feature table on which Log2FC/adj-p differential analysis is performed; also generates the volcano/differential outputs downstream) — https://github.com/NBDZ/dbnorm
- **R** (Statistical environment in which the linear regression per feature, BH correction, and volcano plotting are executed)
- **ggplot2** (R plotting library used to draw the volcano plot of Log2FC vs -log10(adjusted p-value))

## Examples

```
dbnormBer(data); then fit lm per feature, p.adjust(method="BH"), filter(Log2FC > 1.5, adj_p < 0.05), write.csv(sig_features)
```

## Evaluation signals

- All exported significant features satisfy BOTH criteria simultaneously: Log2FC > 1.5 and BH-adjusted p < 0.05.
- Volcano plot axes are Log2FC (x) and -log10(adjusted p-value) (y), with the significant subset visually highlighted.
- Number of significant metabolites in ber-corrected data exceeds the count in raw-data analysis; metabolites significant only after correction (mostly lipids) are captured.
- Input table is log2-scaled before regression (README requires data normalized/scaled on log2 so high-abundance features do not mask technical heterogeneity).
- CSV and PNG outputs are reproducible: the significant-feature list and plot are regenerated from the same corrected matrix without manual curation.

## Limitations

- Applied to the 8-week HFD vs control subcutaneous adipose contrast; the significance criteria were stated for this specific untargeted UPLC-HRMS metabolomics experiment.
- Input must already be batch-corrected and log2-scaled; applying thresholds to raw data misses metabolites (mostly lipids) whose HFD effect is only evident after ber correction.
- Log2FC > 1.5 is a stringent cutoff that excludes moderate (|Log2FC| ≤ 1.5) but statistically significant changes.

## Evidence

- [results] Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level.: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [full_text] ber-corrected data yielded a more exhaustive candidate list for HFD effects than raw data: [evidence span withheld — anchor status: not_found]
- [readme] Data to be uploaded must be normalized and scaled on the log2: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features (variables) by which technical heterogeneity might be overlooked."
- [abstract] dbnorm assigns a score based on adjusted-R2: "This score, which is denoted by adjusted- R squared (adj-R2), ranges from 0 to 1"
