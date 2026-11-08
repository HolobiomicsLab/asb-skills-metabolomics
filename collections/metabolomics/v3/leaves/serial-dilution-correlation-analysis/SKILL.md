---
name: serial-dilution-correlation-analysis
description: Use when evaluating quantification accuracy of an LC-MS1 processing pipeline
  on a serial dilution dataset (e.g., cross-gradient serum/urine tubes A-M with known
  dilution ratios) where expected intensity-vs-dilution proportionality can serve
  as ground truth.
when_to_use_negative:
- Input is already a processed feature table with no linkage to raw spectra or a dilution
  design — rerunning feature detection is unnecessary
- Dataset has no dilution series or other monotonic dose/response structure, so intensity-vs-dilution
  correlation is not a meaningful accuracy metric
- Goal is MS2-based compound identification or DDA/DIA spectral deconvolution rather
  than MS1 quantification performance assessment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnalystR 4.0
  role: Auto-optimized LC-MS1 spectra processing (peak picking, alignment, gap filling,
    annotation) generating the feature table whose intensities are correlated against
    dilution ratios
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: R
  role: Runtime environment and stats package used to compute per-feature Pearson
    correlations and summarize the fraction of features with r > 0.85
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:a2d9be73d09f1975b77552f4bf1823d600e343ea6cbb41f77e63d28ba04bd58f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# serial-dilution-correlation-analysis

## Summary

Quantifies LC-MS1 feature detection accuracy by correlating measured feature intensities against known dilution ratios across a serial dilution series, as used to benchmark MetaboAnalystR 4.0's auto-optimized spectra processing workflow.

## When to use

Use when evaluating quantification accuracy of an LC-MS1 processing pipeline on a serial dilution dataset (e.g., cross-gradient serum/urine tubes A-M with known dilution ratios) where expected intensity-vs-dilution proportionality can serve as ground truth.

## When NOT to use

- Input is already a processed feature table with no linkage to raw spectra or a dilution design — rerunning feature detection is unnecessary
- Dataset has no dilution series or other monotonic dose/response structure, so intensity-vs-dilution correlation is not a meaningful accuracy metric
- Goal is MS2-based compound identification or DDA/DIA spectral deconvolution rather than MS1 quantification performance assessment

## Inputs

- Raw LC-MS1 spectra in centroid mzML format for a serial dilution series (e.g., ST002796, tubes A-M, 3 replicates each)
- Expected dilution ratios per tube derived from the serial dilution design

## Outputs

- Per-feature Pearson correlation coefficients (r) between intensity and dilution ratio
- dilution_pearson_correlations.csv containing per-feature r values and a summary of the fraction of features with r > 0.85
- Complete MS1 feature table across all dilution samples (from peak picking, alignment, gap filling, annotation)

## How to apply

Obtain raw centroid spectra in mzML format covering the serial dilution series (e.g., Metabolomics Workbench accession ST002796, 13 tubes A-M with three replicates per dilution, 11 mixtures plus two pure urine/serum samples). Run the pipeline under evaluation — for MetaboAnalystR 4.0, the auto-optimized LC-MS1 workflow covering peak picking, peak alignment, gap filling, and peak annotation — to generate a complete MS1 feature table across all dilution samples. Assemble the expected dilution ratio for each tube from the experimental design. For every MS1 feature, compute the Pearson correlation between measured intensity and dilution ratio across the 13 dilution points using R's stats package. Judge success by the fraction of features with r > 0.85: MetaboAnalystR 4.0 reported over 90% of features above this threshold, outperforming other tools under default settings.

## Related tools

- **MetaboAnalystR 4.0** (Auto-optimized LC-MS1 spectra processing (peak picking, alignment, gap filling, annotation) generating the feature table whose intensities are correlated against dilution ratios) — https://github.com/xia-lab/MetaboAnalystR
- **R** (Runtime environment and stats package used to compute per-feature Pearson correlations and summarize the fraction of features with r > 0.85)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE); cor(feature_intensities, dilution_ratios, method = "pearson")
```

## Evaluation signals

- More than 90% of detected MS1 features show Pearson correlation coefficients > 0.85 against the expected dilution ratios
- dilution_pearson_correlations.csv contains an r value for every detected feature plus the summary fraction
- Feature table is complete across all 13 tubes (A-M) with three replicates per dilution, with no missing values after gap filling
- Result is consistent with the serial dilution quantification performance reported for MetaboAnalystR 4.0 in the benchmark

## Limitations

- Requires a well-characterized dilution design (11 mixtures plus two pure urine/serum samples for the benchmark); without accurate expected ratios the correlation is meaningless
- Findings apply to LC-MS1 quantification under MetaboAnalystR 4.0's auto-optimized settings; no stable isotope-labeling analysis is supported, and the R environment version is not pinned in the article

## Evidence

- [results] Over 90% of MS1 features detected by MetaboAnalystR on the serial dilution series exhibited high average Pearson correlation coefficients (>0.85): [evidence span withheld — anchor status: not_found]
- [abstract] Validation studies abstract claim: "Comprehensive validation studies using LC-MS1 and MS2 spectra obtained from standards mixtures, dilution series and clinical metabolomics samples have shown its excellent performance"
- [readme] Serial dilutions detection claim: "Serial dilutions demonstrate that MetaboAnalystR 4.0 can accurately detect and identify > 10% more high-quality MS and MS/MS features."
- [readme] Install command grounding: "devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)"
