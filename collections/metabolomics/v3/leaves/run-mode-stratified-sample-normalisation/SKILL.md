---
name: run-mode-stratified-sample-normalisation
description: Use when a metabolomics dataset spans multiple run-mode batches (e.g.
  'platform' or 'run_day' columns with pos/neg values) and samples should be median-normalised
  within each declared batch before downstream QC or filtering.
when_to_use_negative:
- The sample metadata contains no declared run-mode/batch column, or all samples belong
  to a single batch — the normalisation is conditional on a user-declared batch column
  and should not be forced
- The data layer has already been batch-normalised (dest_layer 'batch_normalised'
  exists); re-running would layer normalisations on top of each other
- The goal is only QC filtering (quality_control with sample_missingness/feature_missingness/total_peak_area_sd
  thresholds), not cross-batch scaling
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metaboprep
  role: Provides the Metaboprep S7 object, the batch_normalise() function that performs
    run-mode-stratified normalisation, and the summary()/log reporting used to verify
    the new layer
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime language in which the metaboprep package and batch_normalise call
    are executed
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:d67a526525257c0142f9a9a2840201d44ef335a3be1b84b4a342364c30b71b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# run-mode-stratified-sample-normalisation

## Summary

A metaboprep (R) skill for median-normalising metabolomics samples within declared run-mode batches (e.g. pos/neg platform runs), creating a new batch-normalised data layer while leaving the input layer intact and recording the step in the pipeline log.

## When to use

Use when a metabolomics dataset spans multiple run-mode batches (e.g. 'platform' or 'run_day' columns with pos/neg values) and samples should be median-normalised within each declared batch before downstream QC or filtering.

## When NOT to use

- The sample metadata contains no declared run-mode/batch column, or all samples belong to a single batch — the normalisation is conditional on a user-declared batch column and should not be forced
- The data layer has already been batch-normalised (dest_layer 'batch_normalised' exists); re-running would layer normalisations on top of each other
- The goal is only QC filtering (quality_control with sample_missingness/feature_missingness/total_peak_area_sd thresholds), not cross-batch scaling

## Inputs

- Metaboprep S7 object with an 'input' data layer holding the raw metabolite matrix
- Sample annotation/metadata containing a run-mode batch column (e.g. sample_id, neg, pos, run_day, box_id, lot)
- Raw metabolite data from package extdata (dummy_data.csv / dummy_samples.csv / dummy_features.csv, or metabolon_v1.1_example.xlsx sheet 'OrigScale')

## Outputs

- New data layer 'batch_normalised' inside the Metaboprep object
- Batch-normalised metabolite matrix exported as batch_normalised_data.csv
- Log entry recording that batch normalisation was applied
- Updated object summary listing the added data layer

## How to apply

First construct a Metaboprep S7 object from the metabolite matrix and sample/feature metadata (e.g. extdata files dummy_data.csv, dummy_samples.csv and dummy_features.csv, or read_metabolon on metabolon_v1.1_example.xlsx with sheet='OrigScale') using Metaboprep(data, samples, features). Normalisation is conditional: it is triggered 'when necessary' only because the user declares, via the parameter file, a column name to be used for batch normalisation. Call batch_normalise with run_mode_col naming that sample-metadata column (e.g. 'platform'), run_mode_colmap mapping run-mode labels (e.g. c(pos="pos", neg="neg")), source_layer = "input" and dest_layer = "batch_normalised", so samples are normalised within each declared run-mode batch. Confirm the new layer was created with summary(mydata) — layer names should change from 'input, qc'-style listings to include 'batch_normalised' — and inspect normalised values via mydata@data[..., "batch_normalised"]. Finally export the batch-normalised matrix (e.g. batch_normalised_data.csv) and ensure the log file records the normalisation step, consistent with metaboprep's open-disclosure reporting of batch-variable influence.

## Related tools

- **metaboprep** (Provides the Metaboprep S7 object, the batch_normalise() function that performs run-mode-stratified normalisation, and the summary()/log reporting used to verify the new layer) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime language in which the metaboprep package and batch_normalise call are executed) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- batch_normalise(mydata, run_mode_col = "platform", run_mode_colmap = c(pos="pos", neg="neg"), source_layer = "input", dest_layer = "batch_normalised")
```

## Evaluation signals

- summary(mydata) lists the new 'batch_normalised' layer among Data Layers / Layer Names
- mydata@data[..., "batch_normalised"] returns numeric normalised values with the same dimensions as the input layer
- The log file contains an entry recording that batch normalisation was applied
- Exported batch_normalised_data.csv row/column identifiers match the input metabolite matrix (samples x features preserved)
- Normalisation is computed within each declared run-mode group (e.g. pos and neg separately), not pooled across batches

## Limitations

- Normalisation is applied conditionally ('when necessary') only when the user declares a batch-normalisation column in the parameter file; with no such column the step is skipped
- The metaboprep GitHub repo is marked 'Lifecycle: experimental' and the README documents the v2_development quality_control pipeline but not batch_normalise, so the normalisation interface may change
- The run_mode_colmap must cover every run-mode label present in the metadata column (e.g. both pos and neg), otherwise batches may not be handled as intended

## Evidence

- [other] batch_normalise trigger via parameter file: [evidence span withheld — anchor status: not_found]
- [abstract] batch variable reporting for open disclosure: "A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure"
- [readme] sample metadata run-mode columns: [evidence span withheld — anchor status: external_doc]
- [intro] package purpose and export format: "Read in and processes (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere"
