---
name: batch-correction-analysis
description: Use when raw ion concentrations (e.g. ICP-MS ppm values) span multiple
  measurement batches and you need to remove batch effects prior to standardisation,
  clustering, or network inference on element profiles.
when_to_use_negative:
- Input data is already normalised/standardised — IonFlow can process pre-normalised
  data directly for network extraction, so re-applying median batch correction would
  double-correct
- Data comes from a single batch with no replicated control strain, where per-ion-per-batch
  median scaling has no between-batch variation to remove
- The goal is downstream analysis only (PCA, hierarchical clustering, relevance networks,
  GO enrichment) — use those IonFlow sections directly on an already-cleaned dataset
edam_operation: http://edamontology.org/operation_3432
edam_topics: []
tools:
- name: IonFlow
  role: R package whose PreProcessing function implements log-transformation followed
    by median batch correction (method_norm/control_use/control_lines) before standardisation
    and symbolisation
  repo: https://github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web platform exposing IonFlow's batch-correction/pre-processing workflow to
    non-command-line users
  repo: https://github.com/galaxyproject/galaxy
- name: MetaboFlow
  role: Parent package from which the IonFlow R package was modified; hosts the Pre-processing
    section and the IonFlow R library
  repo: https://github.com/AlinaPeluso/MetaboFlow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:ba37d150d6c711d377cca7b0f52640d001eb4129398e6fc0e1928954aad7b668
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wanchanglin/ionflow
  tools:
  - Galaxy
  - IonFlow
  - MetaboFlow
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# batch-correction-analysis

## Summary

Batch correction of ICP-MS ionomics data as implemented in IonFlow: after log-transformation, ion concentrations are scaled to per-ion, per-batch medians (optionally anchored to a control strain such as YDL227C) to remove between-batch variability before standardisation and symbolic profiling.

## When to use

Use when raw ion concentrations (e.g. ICP-MS ppm values) span multiple measurement batches and you need to remove batch effects prior to standardisation, clustering, or network inference on element profiles.

## When NOT to use

- Input data is already normalised/standardised — IonFlow can process pre-normalised data directly for network extraction, so re-applying median batch correction would double-correct
- Data comes from a single batch with no replicated control strain, where per-ion-per-batch median scaling has no between-batch variation to remove
- The goal is downstream analysis only (PCA, hierarchical clustering, relevance networks, GO enrichment) — use those IonFlow sections directly on an already-cleaned dataset

## Inputs

- Raw ionomics data frame of ICP-MS ion concentrations in ppm (e.g. 14 ions: Ca44, Cd111, Co59, Cu65, Fe56, K39, Mg25, Mn55, Mo95, Na23, Ni60, P31, S34, Zn66 for 1454 single-gene haploid yeast knockouts)
- Optional pre-defined ion standard deviations (stdev) computed from control samples/genes
- Batch metadata and control-line specification (control_use / control_lines parameters)

## Outputs

- Batch-corrected log-transformed dataset summarised in stats.median_batch_corrected_data
- data.line.zscores (line-level z-score table) and data.line.symb (symbolic profiles at thres_symb)
- stats tables (stats.raw_data, stats.outliers) and diagnostic plots

## How to apply

Run the PreProcessing stage of the IonFlow R package: first log-transform the raw concentration values, then scale each ion to the median taken for that ion within each batch, driven by the method_norm, control_use, or control_lines parameters. A control sample measured in every batch (e.g. the YDL227C knock-out control strain in the S. cerevisiae ionome dataset) can anchor the correction via control_use/control_lines. Outlier removal (log.FC.distance, mad, or IQR methods with thres_outl; outer fences defined as Q1 - 3*IQ and Q3 + 3*IQ) precedes or accompanies the correction to prevent unreliable samples distorting batch medians. Corrected data then feed the standardisation step (std, mad, or custom scaling, optionally using pre-defined standard deviations from control genes) and symbolic profile extraction at threshold thres_symb.

## Related tools

- **IonFlow** (R package whose PreProcessing function implements log-transformation followed by median batch correction (method_norm/control_use/control_lines) before standardisation and symbolisation) — https://github.com/wanchanglin/ionflow
- **Galaxy** (Web platform exposing IonFlow's batch-correction/pre-processing workflow to non-command-line users) — https://github.com/galaxyproject/galaxy
- **MetaboFlow** (Parent package from which the IonFlow R package was modified; hosts the Pre-processing section and the IonFlow R library) — https://github.com/AlinaPeluso/MetaboFlow

## Examples

```
pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd); pre_proc$stats.median_batch_corrected_data
```

## Evaluation signals

- stats.median_batch_corrected_data shows per-ion median exactly 0 (all 14 ions have median 0 in the yeast benchmark) with reduced variance relative to stats.raw_data
- Corrected distributions bounded within plausible log-scale ranges (e.g. Ca: min -4.311, max 1.671 in the benchmark) rather than raw ppm extremes (Ca max 902.568 ppm)
- Downstream outputs data.line.zscores and data.line.symb are produced with no missing values, matching the raw data's zero-missing invariant
- Outlier counts in stats.outliers per ion (e.g. 305/9999 for Ca, 0.22%) reflect the Q1 - 3*IQ / Q3 + 3*IQ outer-fence rule applied consistently
- Batch-to-batch median drift of log-transformed concentrations collapses to zero after correction when the control strain YDL227C is used for anchoring

## Limitations

- No version number stated for IonFlow or Galaxy, complicating reproducibility of the correction pipeline
- No quantitative benchmark results or performance metrics reported for the batch correction on the S. cerevisiae or HeLa case studies
- Standard deviations used for standardisation are optionally user-supplied; if omitted they are recomputed from the data, which may interact with the batch correction outcome
- Correction is defined per ion within each batch, so ions with very few non-outlier observations in a batch may yield unstable medians

## Evidence

- [readme] Median batch correction of IonFlow: [evidence span withheld — anchor status: external_doc]
- [readme] Control strain anchoring: [evidence span withheld — anchor status: external_doc]
- [methods] Batch-correction parameters: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline context: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
