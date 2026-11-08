---
name: fold-change-pvalue-extraction-from-peak-tables
description: Use when you have WiPP's merged, classified peak table for a two-condition
  GC-MS dataset and need per-feature fold change and p-value to compare against a
  previously published feature list.
when_to_use_negative:
- You only have raw GC-MS data and have not yet run WiPP's peak picking and result
  integration — this skill operates on the merged final peak table, not raw mzML/NetCDF
  files.
- The dataset has more than two experimental conditions or no predefined two-group
  comparison — fold change and p-value extraction as described assumes a two-condition
  contrast.
- Features lack WiPP annotation (no retention-index support or no class assignment)
  — statistics on unannotated features cannot be cross-referenced against a published
  analyte table.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: WiPP
  role: Produces the final high-quality peak table (peak classification merged with
    results from different peak detection algorithms) that the fold change and p-value
    are extracted from
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Environment in which WiPP runs and in which the extraction and CSV record-writing
    is scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:415a512e79f8de3d8b786b55fbaa107762159097dfa4d382a86d2f8e400c2210
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fold-change-pvalue-extraction-from-peak-tables

## Summary

Extracts condition-wise fold changes (HCC vs cirrhosis) and p-values for an annotated GC-MS feature (e.g. a hexose) from the final high-quality peak table produced by WiPP's result integration, and records it for cross-checking against the original published analysis.

## When to use

Use when you have WiPP's merged, classified peak table for a two-condition GC-MS dataset and need per-feature fold change and p-value to compare against a previously published feature list.

## When NOT to use

- You only have raw GC-MS data and have not yet run WiPP's peak picking and result integration — this skill operates on the merged final peak table, not raw mzML/NetCDF files.
- The dataset has more than two experimental conditions or no predefined two-group comparison — fold change and p-value extraction as described assumes a two-condition contrast.
- Features lack WiPP annotation (no retention-index support or no class assignment) — statistics on unannotated features cannot be cross-referenced against a published analyte table.

## Inputs

- Final high-quality peak table from WiPP result integration (peak classification merged across peak detection algorithms)
- Feature annotations assigned during WiPP annotation (retention-index-supported where alkane-containing wash samples exist)
- Sample grouping of the two experimental conditions (e.g. HCC and cirrhosis plasma samples)

## Outputs

- Single-row structured record (CSV) containing feature annotation, fold change (HCC vs cirrhosis), and p-value
- Comparison statement identifying which features are WiPP-specific findings versus those reported in the original study

## How to apply

Start from the final high-quality peak set that WiPP produces by merging the classifier's quality information with results from multiple embedded peak picking algorithms (peak classification output). Restrict to features whose annotation is supported by the WiPP annotation stage — for the HCC vs cirrhosis plasma GC-MS case this is a hexose annotated via retention index where alkanes in wash samples are available. Compute the HCC-vs-cirrhosis fold change and p-value from the two experimental condition groups in the peak table. Cross-reference the feature and its statistics against the corresponding table of the original published study (Table 1 of Ranjbar et al. for the HCC case) to confirm whether it is a WiPP-specific finding. Emit a single-row structured record (annotation, fold change, p-value, e.g. hexose_feature_record.csv) so the output can be reviewed by a domain expert.

## Related tools

- **WiPP** (Produces the final high-quality peak table (peak classification merged with results from different peak detection algorithms) that the fold change and p-value are extracted from) — https://github.com/bihealth/WiPP
- **Python 3** (Environment in which WiPP runs and in which the extraction and CSV record-writing is scripted)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('WiPP_final_peak_table.csv'); row = df[(df['annotation']=='hexose') & (df['group']=='HCC_vs_cirrhosis')]; row[['annotation','fold_change','p_value']].to_csv('hexose_feature_record.csv', index=False)"
```

## Evaluation signals

- The extracted fold change and p-value for the annotated feature should match a manual recomputation from the peak table for the same condition grouping.
- The feature should be confirmed absent from the original study's Table 1 (Ranjbar et al.) to be classified as a WiPP-specific finding.
- The output record should contain exactly one row per target feature with three fields: annotation, fold change, p-value (schema check on hexose_feature_record.csv).
- Fold change direction must be consistent with the condition grouping (HCC relative to cirrhosis).
- Validation via expert_review: a domain expert confirms annotation, fold change, and p-value against the original study's Table 1.

## Limitations

- Statistics are only as good as the underlying peak table; WiPP notes that medium- and low-quality peaks are kept for further inspection, so misclassification of peak quality could affect the extracted statistics.
- The example project uses only 25 annotated peaks per algorithm, producing a classifier that is 'likely not to be accurate' — a weak classifier upstream could propagate errors into the final peak table.
- WiPP v1.0 is not supported on Ubuntu 18 (Bionic Beaver) due to lacking support of incorporated R packages, which constrains where the extraction pipeline can run.

## Evidence

- [abstract] final high-quality peak set for extraction: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [abstract] medium/low peaks retained: "Medium- and low-quality peaks are kept for further inspection."
- [readme] peak picking and integration step: "Many sequential substeps are in fact happening during this final step of the pipeline such as classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection"
- [readme] annotation via wash alkanes: "Here, we assume that these samples contain the alkanes that are used for Retention Index calculation."
- [readme] input file formats: "**WiPP** supports mzML, mzData and NetCDF file formats."
