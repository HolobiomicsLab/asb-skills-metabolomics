---
name: metabolomics-replicate-qc-evaluation
description: Use when raw LC-MS spectra from a serial dilution series (known dilution
  design, replicate tubes) are available and you need to validate that a spectra-processing
  workflow quantifies features proportionally to concentration.
when_to_use_negative:
- The input is already a processed feature table with no linked raw dilution-series
  spectra and no known dilution ratios, so correlations against expected concentrations
  cannot be computed.
- The dataset is a single-condition or case-control study with no dilution/concentration
  gradient — replicate QC by correlation to dilution ratio is undefined.
- The goal is MS2 compound-identification QC (e.g. DDA/DIA deconvolution or spectral
  library matching accuracy) rather than MS1 quantification accuracy.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: Runs the auto-optimized LC-MS1 workflow (peak picking, alignment, gap filling,
    annotation) that generates the MS1 feature tables being evaluated on the dilution
    series
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Environment in which the processing workflow runs and in which per-feature
    Pearson correlations against dilution ratios are computed with the stats package
- name: OptiLCMS
  role: Companion package installed alongside MetaboAnalystR that underpins the LC-MS1
    auto-optimized feature detection and quantification module
  repo: https://github.com/xia-lab/OptiLCMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:ee8fa435611c3f021b0a123c12de7940878413b2a5d754a0dbf6b1ceff76106d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolomics-replicate-qc-evaluation

## Summary

Evaluates LC-MS1 feature quantification accuracy by computing Pearson correlations between measured feature intensities and known dilution ratios across a serial dilution series, then judging the processing workflow by the fraction of features exceeding a correlation threshold.

## When to use

Use when raw LC-MS spectra from a serial dilution series (known dilution design, replicate tubes) are available and you need to validate that a spectra-processing workflow quantifies features proportionally to concentration.

## When NOT to use

- The input is already a processed feature table with no linked raw dilution-series spectra and no known dilution ratios, so correlations against expected concentrations cannot be computed.
- The dataset is a single-condition or case-control study with no dilution/concentration gradient — replicate QC by correlation to dilution ratio is undefined.
- The goal is MS2 compound-identification QC (e.g. DDA/DIA deconvolution or spectral library matching accuracy) rather than MS1 quantification accuracy.

## Inputs

- Centroid mzML raw spectra files for all dilution tubes and replicates (e.g. 13 tubes A–M, three replicates each, Metabolomics Workbench ST002796)
- Serial dilution design table mapping each tube to its expected dilution ratio (11 mixtures plus two pure urine/serum samples)

## Outputs

- Per-feature Pearson correlation coefficients between dilution ratio and intensity
- Summary fraction of MS1 features with correlation coefficient > 0.85
- dilution_pearson_correlations.csv containing all per-feature r values plus the summary

## How to apply

Obtain centroid mzML raw spectra covering all dilution points (e.g. the 13-tube A–M serum/urine series from Metabolomics Workbench accession ST002796, with three replicates per dilution) and process them with the MetaboAnalystR 4.0 auto-optimized LC-MS1 workflow (peak picking, alignment, gap filling, annotation) to build a complete feature table. Assemble the expected dilution ratios for each tube from the serial dilution design, distinguishing the 11 mixtures from the two pure urine/serum endpoints. For every MS1 feature, compute the Pearson correlation between measured intensity and dilution ratio across the 13 dilution points using R's stats package, and write per-feature r values to dilution_pearson_correlations.csv. Accept the evaluation when more than 90% of detected features show r > 0.85, the benchmark MetaboAnalystR 4.0 achieved against other tools under default settings.

## Related tools

- **MetaboAnalystR 4.0** (Runs the auto-optimized LC-MS1 workflow (peak picking, alignment, gap filling, annotation) that generates the MS1 feature tables being evaluated on the dilution series) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Environment in which the processing workflow runs and in which per-feature Pearson correlations against dilution ratios are computed with the stats package)
- **OptiLCMS** (Companion package installed alongside MetaboAnalystR that underpins the LC-MS1 auto-optimized feature detection and quantification module) — https://github.com/xia-lab/OptiLCMS

## Examples

```
Rscript -e 'devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE); corr <- cor(feature_intensities, dilution_ratios); sum(corr > 0.85)/length(corr)'
```

## Evaluation signals

- More than 90% of detected MS1 features show Pearson correlation coefficient > 0.85 between dilution ratios and feature intensities across the 13 dilution points.
- dilution_pearson_correlations.csv exists and contains a per-feature r value for every detected MS1 feature plus the summary fraction above threshold.
- The feature table is complete across all 13 tubes (11 mixtures plus two pure urine/serum samples) with three replicates per dilution — no missing tubes in the peak table.
- Results match or exceed the reported MetaboAnalystR 4.0 serial dilution performance relative to other tools run under default settings.

## Limitations

- The >90% / r > 0.85 acceptance criterion is calibrated to the MetaboAnalystR 4.0 benchmark; other tools may require the same default-settings comparison to be a fair benchmark.
- Performance depends on the dilution design being correctly encoded — the two pure urine/serum endpoints must be separated from the 11 mixture tubes when computing expected ratios.
- Quantification accuracy for MS1 features does not extend the validation to MS2-based compound identification, which depends on the availability of reference spectra databases.

## Evidence

- [results] Over 90% of MS1 features detected by MetaboAnalystR on the serial dilution series exhibited high average Pearson correlation coefficients (>0.85) between dilution ratios and feature intensities: [evidence span withheld — anchor status: not_found]
- [abstract] an auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing: "an auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing"
- [readme] Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features.: "Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features."
- [methods] devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE): "devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)"
