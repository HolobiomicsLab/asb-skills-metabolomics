---
name: cross-study-feature-comparison-literature-check
description: 'Use when a pipeline re-run on a public dataset must be validated against
  the original publication: you have annotated, quality-filtered features (annotation,
  fold change, p-value) and need to check which ones the original study did or did
  not report.'
when_to_use_negative:
- Input is already a curated feature table whose features you do not need to compare
  against any published reference — skip the literature check and proceed to downstream
  statistics.
- The dataset has no published reference study (no Table/analyte list to cross-reference);
  the comparison step has nothing to validate against.
- The peak table has not yet been through result integration (classifier quality still
  unmapped to individual peaks), so annotations and statistics do not reflect the
  final high-quality peak set.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: WiPP
  role: Produces the final high-quality peak table and per-feature annotations that
    feed the cross-study comparison and literature check.
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Environment for the filtering/statistical-extraction and CSV record-writing
    scripts used in the comparison workflow.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:1e110f76e88960f93b046f89c418e896845b0711b88bae7c5787bf53329d2454
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cross-study-feature-comparison-literature-check

## Summary

Compare a reprocessed metabolomics feature table against a published reference study to identify novel significantly different features that the original analysis missed. In the WiPP paper this means extracting annotated features (e.g. a hexose) from the final high-quality peak set and cross-referencing them against the original article's reported analytes table.

## When to use

Use when a pipeline re-run on a public dataset must be validated against the original publication: you have annotated, quality-filtered features (annotation, fold change, p-value) and need to check which ones the original study did or did not report.

## When NOT to use

- Input is already a curated feature table whose features you do not need to compare against any published reference — skip the literature check and proceed to downstream statistics.
- The dataset has no published reference study (no Table/analyte list to cross-reference); the comparison step has nothing to validate against.
- The peak table has not yet been through result integration (classifier quality still unmapped to individual peaks), so annotations and statistics do not reflect the final high-quality peak set.

## Inputs

- Final high-quality peak table from WiPP's result integration step (classifier quality merged across peak detection algorithms) for the full dataset (mzML / mzData / NetCDF-derived samples organized by condition)
- Per-feature annotation labels from the WiPP annotation stage (e.g. 'hexose'), retention-index-supported when alkanes in Wash samples are available
- Statistical comparison of the two experimental condition groups: fold change (e.g. HCC vs cirrhosis) and p-value per feature
- Reference table from the original published study listing its reported analytes (e.g. Table 1 of Ranjbar et al.)

## Outputs

- Structured record per novel feature: annotation, fold change, p-value (e.g. hexose_feature_record.csv)
- List of confirmed features shared with the original study, with fold-change/p-value comparison to the published values
- Expert-review validation flag for each cross-study comparison record

## How to apply

Load the final high-quality peak table produced by the result integration step, where classifier quality information has been merged with results from the different peak detection algorithms on the full dataset. Filter the table for the feature class of interest using annotations assigned during the WiPP annotation stage — retention-index-supported annotations where alkane-containing wash samples were available for RI calculation. Extract the fold change (e.g. HCC vs cirrhosis) and p-value for each candidate feature from the statistical comparison of the two experimental condition groups. Cross-reference the annotated features against the reference table of the original publication (e.g. Table 1 of Ranjbar et al.) to classify each feature as confirmed (present with comparable fold change and p-value) or novel (not reported in the original study). Emit a single-row structured record per novel feature (annotation, fold change, p-value) to a CSV such as hexose_feature_record.csv, and submit it for expert review against the original table as validation. In the HCC vs cirrhosis plasma GC-MS case study, this procedure surfaced one hexose not reported by the original study while confirming six of nine validated analytes.

## Related tools

- **WiPP** (Produces the final high-quality peak table and per-feature annotations that feed the cross-study comparison and literature check.) — https://github.com/bihealth/WiPP
- **Python 3** (Environment for the filtering/statistical-extraction and CSV record-writing scripts used in the comparison workflow.)

## Examples

```
../../run_WiPP.sh pp -n 4
```

## Evaluation signals

- Each novel-feature record contains exactly the fields annotation, fold change, and p-value, written as one row to hexose_feature_record.csv.
- The novel feature's annotation is not present in the original study's Table 1 (e.g. the hexose was absent from Ranjbar et al.'s reported analytes).
- Confirmed features reproduce the original study: fold changes and p-values comparable to the published values — in the case study, six of nine validated analytes matched.
- A domain expert has reviewed the record against the original Table 1 and confirmed the hexose is a WiPP-specific finding.
- Statistics were computed only on the integrated high-quality peak table, not on raw per-algorithm peak lists.

## Limitations

- Peak annotation via the visualization tool is time-consuming (typically 1200 peaks per algorithm) and only needs doing once per instrument/protocol — the trained SVM classifier can be reused for datasets from the same instrument and acquisition method.
- WiPP was tested on Ubuntu 16 and CentOS 7.6.1810; Ubuntu 18 (Bionic Beaver) is not supported due to lacking support of incorporated R packages.
- Retention-index-supported annotations require optional Wash/blank samples containing alkanes (alkane list, e.g. c10..c34); without them annotation support is weaker.
- A classifier trained on too few annotated peaks will be inaccurate; reuse a provided or previously trained classifier when the annotation set is small.

## Evidence

- [abstract] final high-quality peak set merged across algorithms: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set for immediate"
- [results] case-study result: one hexose not reported in the original study: [evidence span withheld — anchor status: not_found]
- [results] validation against original study: [evidence span withheld — anchor status: not_found]
- [methods] result integration step of the pipeline: "classifier hyperparameter optimisation, peak detection algorithms parameter optimisation, peak detection on the full dataset, output classification and result integration"
- [readme] wash samples used for retention index calculation: "The `Wash` directory is optional but if present should contain the wash or blank sample files. Here, we assume that these samples contain the alkanes that are used for Retention Index calculation."
