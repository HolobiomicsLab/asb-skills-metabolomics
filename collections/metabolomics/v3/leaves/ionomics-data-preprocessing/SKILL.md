---
name: ionomics-data-preprocessing
description: Use when raw ICP-MS ionomics concentration data (element ppm values per
  sample/knockout) must be cleaned of batch effects and unreliable samples, and converted
  into line-level z-score and symbolic profiles before exploratory, clustering, or
  network analysis.
when_to_use_negative:
- Input data are already normalised and standardised — IonFlow can directly process
  normalised/standardised data for network extraction and element-element correlation
  analysis, bypassing this pre-processing
- The goal is a downstream step (PCA, hierarchical clustering, relevance network extraction,
  GO enrichment, network analysis) rather than producing the cleaned dataset that
  these sections require
- Data are metabolomics rather than ICP-MS ionome measurements, unless applicability
  has been separately validated
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: IonFlow
  role: R package providing the PreProcessing function (log-transform, median batch
    correction, outlier detection, standardisation, symbolisation)
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web platform wrapping IonFlow as a user-friendly tool; installed via tool_conf.xml
    or the Galaxy Tool Shed
  repo: https://github.com/galaxyproject/galaxy
- name: R
  role: Language in which the IonFlow package and PreProcessing pipeline are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:e4d612f6a4fbe6ecd896f76173e9979a833dc66c3dd9c2815e4ec9af1d5e8f3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ionomics-data-preprocessing

## Summary

Pre-processing of raw ICP-MS ionomics data (e.g. 14 ion concentrations across yeast single-gene knockouts) using IonFlow's PreProcessing function, which log-transforms, batch-corrects to per-ion/per-batch medians, removes extreme outliers via interquartile outer fences, standardises, and symbolises profiles.

## When to use

Use when raw ICP-MS ionomics concentration data (element ppm values per sample/knockout) must be cleaned of batch effects and unreliable samples, and converted into line-level z-score and symbolic profiles before exploratory, clustering, or network analysis.

## When NOT to use

- Input data are already normalised and standardised — IonFlow can directly process normalised/standardised data for network extraction and element-element correlation analysis, bypassing this pre-processing
- The goal is a downstream step (PCA, hierarchical clustering, relevance network extraction, GO enrichment, network analysis) rather than producing the cleaned dataset that these sections require
- Data are metabolomics rather than ICP-MS ionome measurements, unless applicability has been separately validated

## Inputs

- Raw ionomics data frame of ICP-MS element concentrations (ppm, e.g. Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66), optical-density adjusted
- Optional: pre-defined set of ion standard deviations computed from control-gene/strain measurements

## Outputs

- Cleaned pre-processed dataset for downstream sections
- data.line.zscores (line-level z-score profiles)
- data.line.symb (symbolic profiles at threshold thres_symb)
- stats.raw_data, stats.median_batch_corrected_data, stats.outliers summary tables
- Diagnostic plots

## How to apply

Load the raw data frame of ion concentrations (optionally with a pre-defined set of ion standard deviations computed from control strains such as knock-out YDL227C, the control measured in every batch). Run the IonFlow PreProcessing function, which first log-transforms concentration values, then scales the data to the median taken for each ion within each batch (median batch correction). Outlier detection uses lower outer fence Q1 - 3*IQ and upper outer fence Q3 + 3*IQ (Q1/Q3 = first/third quantile), treating points beyond the outer fences as extreme outliers; per-ion outlier counts and percentages are reported in stats.outliers. Standardisation uses either the user-supplied standard deviations or standard deviations computed from the data, and symbolic profiles are extracted at threshold thres_symb. The function must be run first because it produces the cleaned dataset used by all other IonFlow sections (exploratory analysis, clustering, network analysis).

## Related tools

- **IonFlow** (R package providing the PreProcessing function (log-transform, median batch correction, outlier detection, standardisation, symbolisation)) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Web platform wrapping IonFlow as a user-friendly tool; installed via tool_conf.xml or the Galaxy Tool Shed) — https://github.com/galaxyproject/galaxy
- **R** (Language in which the IonFlow package and PreProcessing pipeline are implemented)

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); data(IonData); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- stats.raw_data table reproduces per-ion min/quartiles/median/mean/max/variance for the 14 measured ions; e.g. Fe median 7.295 ppm, K median 2495.265 ppm
- After median batch correction, the median of each ion's distribution equals 0 (see stats.median_batch_corrected_data, e.g. Ca median 0, Cd median 0)
- stats.outliers reports per-ion outlier counts and percentages consistent with the Q1 - 3*IQ / Q3 + 3*IQ outer-fence rule; e.g. Ca: 305 outliers (0.22%), Mg: 8 outliers (0.01%)
- No missing values should remain in the cleaned dataset before downstream sections run
- Output includes data.line.zscores and data.line.symb tables plus diagnostic plots

## Limitations

- Outlier removal is ion-wise via outer-fence (IQR) rules; the README does not document thres_outl, mad, or log.FC.distance alternatives in the visible text, so per-ion outlier rates are small (0.01–0.22%)
- Pre-defined ion standard deviations are optional; if not supplied, standard deviations computed from the data are used for standardisation, which may differ from control-derived scaling
- The paper does not report quantitative benchmark results or performance metrics for the two case-study datasets, and no version number for IonFlow is stated

## Evidence

- [readme] pre_proc <- PreProcessing(data=IonData,stdev=pre_defined_sd): [evidence span withheld — anchor status: external_doc]
- [readme] outer fence rule: [evidence span withheld — anchor status: external_doc]
- [readme] median batch correction: [evidence span withheld — anchor status: external_doc]
- [readme] pre-processing produces cleaned dataset: [evidence span withheld — anchor status: external_doc]
- [abstract] IonFlow processes raw data: "designed as a pipeline that can process raw data to enable exploration and interpretation"
