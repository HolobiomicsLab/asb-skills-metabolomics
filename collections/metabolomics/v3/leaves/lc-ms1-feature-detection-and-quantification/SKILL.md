---
name: lc-ms1-feature-detection-and-quantification
description: Use when starting from raw, centroided LC-MS1 spectra (e.g. mzML files)
  and you need peak picking, peak alignment, gap filling, and peak annotation to produce
  a full MS1 feature table for quantification or dilution-series correlation analysis.
when_to_use_negative:
- Input is already a processed feature table — use MetaboAnalystR's statistical analysis
  or functional interpretation modules instead
- The data are MS2 spectra (DDA/DIA) requiring deconvolution or compound identification
  — use PerformDDADeconvolution / PerformDIADeconvolution workflows instead
- A tool with manually tuned, dataset-specific parameters is preferred — auto-optimization
  targets users who lack deep knowledge for parameter settings
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Provides the auto-optimized LC-MS1 feature detection and quantification workflow
    (peak picking, alignment, gap filling, annotation) in R
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: OptiLCMS
  role: LC-MS1 processing dependency installed alongside MetaboAnalystR 4.0
  repo: https://github.com/xia-lab/OptiLCMS
- name: R
  role: Open-source environment hosting the workflow; R stats package used to compute
    per-feature Pearson correlations against dilution ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:f4e6ec953ef4b90fb52cd20565857995c403a33bb22d8083bc2a5153fb115bc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - OptiLCMS
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lc-ms1-feature-detection-and-quantification

## Summary

Process raw LC-MS1 spectra into a complete feature-intensity table using MetaboAnalystR 4.0's auto-optimized feature detection and quantification algorithm, which improves quantification accuracy over default settings of other raw spectral processing tools.

## When to use

Use when starting from raw, centroided LC-MS1 spectra (e.g. mzML files) and you need peak picking, peak alignment, gap filling, and peak annotation to produce a full MS1 feature table for quantification or dilution-series correlation analysis.

## When NOT to use

- Input is already a processed feature table — use MetaboAnalystR's statistical analysis or functional interpretation modules instead
- The data are MS2 spectra (DDA/DIA) requiring deconvolution or compound identification — use PerformDDADeconvolution / PerformDIADeconvolution workflows instead
- A tool with manually tuned, dataset-specific parameters is preferred — auto-optimization targets users who lack deep knowledge for parameter settings

## Inputs

- Raw centroided LC-MS1 spectra in mzML format (e.g. serial dilution tubes A–M, three replicates per dilution, from Metabolomics Workbench accession ST002796)
- Expected dilution ratios for each tube, derived from the serial dilution design (11 mixtures plus two pure urine/serum samples)

## Outputs

- Complete MS1 feature-intensity table across all samples (features x samples) after peak picking, alignment, gap filling, and annotation
- Per-feature Pearson correlation coefficients between intensity and dilution ratio, plus a summary of the fraction of features with r > 0.85 (e.g. written to dilution_pearson_correlations.csv)

## How to apply

Install MetaboAnalystR 4.0 (and its OptiLCMS dependency) from GitHub, then run the auto-optimized LC-MS1 workflow — peak picking, peak alignment, gap filling, and peak annotation — on centroid mzML files to generate a complete feature table across all samples. The auto-optimization removes the need for deep parameter-setting expertise, which the paper identifies as a primary bottleneck in raw spectral processing. For a serial dilution dataset (e.g. Metabolomics Workbench ST002796: 13 tubes A–M, three replicates each, cross-gradient serum/urine), assemble expected dilution ratios and compute, for every MS1 feature, the Pearson correlation between measured intensity and dilution ratio across the 13 dilution points. Judge success by the fraction of features with r > 0.85 — the paper reports >90% of MetaboAnalystR-detected MS1 features exceed this threshold, better than XCMS, MS-DIAL, MZmine, and asari under default settings.

## Related tools

- **MetaboAnalystR 4.0** (Provides the auto-optimized LC-MS1 feature detection and quantification workflow (peak picking, alignment, gap filling, annotation) in R) — https://github.com/xia-lab/MetaboAnalystR
- **OptiLCMS** (LC-MS1 processing dependency installed alongside MetaboAnalystR 4.0) — https://github.com/xia-lab/OptiLCMS
- **R** (Open-source environment hosting the workflow; R stats package used to compute per-feature Pearson correlations against dilution ratios)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE); devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- More than 90% of detected MS1 features show Pearson correlation coefficient > 0.85 between dilution ratio and measured intensity across the 13 dilution points
- The feature table is complete across all samples after gap filling (no missing values blocking per-feature correlation computation)
- Quantification accuracy outperforms XCMS, MS-DIAL, MZmine, and asari run under default settings
- MetaboAnalystR 4.0 detects and identifies > 10% more high-quality MS and MS/MS features than alternatives on serial dilution data

## Limitations

- Integrated LC-MS1 and MS2 spectra processing remains a primary bottleneck in terms of computational efficiency as well as the requirement of deep knowledge for parameter settings (auto-optimization mitigates, but does not eliminate, this)
- No version pinned for the R environment used; README recommends R base > 3.6.1 and notes compatibility of latest R (v4.0.0) is under evaluation
- Installation requires system dependencies (libcairo2-dev, libnetcdf-dev, libxml2, libxt-dev, libssl-dev on Ubuntu; Rtools on Windows; Xcode and GNU Fortran on Mac), and LaTeX issues may force installing without vignettes/manuals

## Evidence

- [abstract] an auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing: "an auto-optimized feature detection and quantification algorithm for LC-MS1 spectra processing"
- [readme] Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features.: "Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features."
- [methods] devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE): "devtools::install_github("xia-lab/OptiLCMS", build = TRUE, build_vignettes = FALSE)"
- [abstract] Comprehensive validation studies using LC-MS1 and MS2 spectra obtained from standards mixtures, dilution series and clinical metabolomics samples have shown its excellent performance across a wide: "Comprehensive validation studies using LC-MS1 and MS2 spectra obtained from standards mixtures, dilution series and clinical metabolomics samples have shown its excellent performance across a wide"
