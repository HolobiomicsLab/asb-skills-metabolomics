---
name: qc-pool-batch-stratification
description: Use when you have pooled-QC injections spread over multiple batches,
  track user-defined m/z–RT pairs with QC4Metabolomics, and need to compare within-batch
  RT/m/z deviations against interbatch deviations to set drift tolerance.
when_to_use_negative:
- When the QC pool has not yet been injected across more than one batch (only within-batch
  variation is estimable).
- When monitoring contaminants rather than user-defined compound m/z–RT pairs (module_Contaminants
  is the relevant module, not module_TrackCmp).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: Shiny app / R package whose module_TrackCmp monitors user-defined m/z–RT pairs
    to extract RT deviation data for the QC pool
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Runtime environment for the QC4Metabolomics package used to group deviations
    by batch and compute the within-/between-batch comparison
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web framework hosting the dashboard that presents the batch-stratified deviation
    results
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment container for QC4Metabolomics so the analysis environment is OS-agnostic
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:97d28b4c1de7d37a1295436177f073e6a12fe51fbcb38b2fa0cdbb2205f84dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# qc-pool-batch-stratification

## Summary

Compare monitored-compound deviations within a single acquisition batch versus across batches for a long-term QC pool (e.g., MetNexs) to quantify which QC regime (short-term or interbatch) dominates drift.

## When to use

Use when you have pooled-QC injections spread over multiple batches, track user-defined m/z–RT pairs with QC4Metabolomics, and need to compare within-batch RT/m/z deviations against interbatch deviations to set drift tolerance.

## When NOT to use

- When the QC pool has not yet been injected across more than one batch (only within-batch variation is estimable).
- When monitoring contaminants rather than user-defined compound m/z–RT pairs (module_Contaminants is the relevant module, not module_TrackCmp).

## Inputs

- MetNexs long-term QC pool retention time and m/z deviation data analyzed within all batches
- User-defined list of m/z and retention time (RT) pairs for module_TrackCmp
- Batch identifier / batch label per injection

## Outputs

- Per-compound RT deviations labeled with batch identifiers
- Two-regime summary comparison (within-batch vs interbatch deviation magnitudes)
- Structured table of deviations for short-term vs interbatch regimes

## How to apply

Load the RT deviation data for the long-term QC pool (MetNexs) tracked by module_TrackCmp from a user-defined list of m/z and RT pairs. Assign each injection to its acquisition batch, grouping the RT deviations by batch identifier so within-batch and between-batch comparisons can be made. Compute typical within-batch deviation magnitude per monitored compound, then compute between-batch deviations by comparing batch-level RT offsets against reference RTs. Verify the two-regime result: within-batch RT deviations normally < 0.05 min, while differences between different batches can be at least twice as large. Save per-compound RT deviations with batch labels and the two-regime summary as a structured table.

## Related tools

- **QC4Metabolomics** (Shiny app / R package whose module_TrackCmp monitors user-defined m/z–RT pairs to extract RT deviation data for the QC pool) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Runtime environment for the QC4Metabolomics package used to group deviations by batch and compute the within-/between-batch comparison) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web framework hosting the dashboard that presents the batch-stratified deviation results) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment container for QC4Metabolomics so the analysis environment is OS-agnostic) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
QC4Metabolomics::module_TrackCmp(mz_rt_pairs, qc_pool_files)  # then group RT deviations by batch
```

## Evaluation signals

- Within-batch RT deviations are normally less than 0.05 min (verify per-compound distribution).
- Differences between different batches are at least twice as large as within-batch deviations.
- Every monitored compound in the QC pool has both a within-batch and a between-batch deviation estimate.
- Output table includes batch labels for each injection so the two regimes can be separated.

## Limitations

- The 0.05 min threshold is empirical for the MetNexs long-term QC pool and the reported instrument setup; it is not a universal tolerance.
- The result is reported for retention time deviations; m/z or intensity deviations may not follow the same two-regime pattern.
- Calibration problems (e.g., calibrant signal loss from a malfunctioning calibrant pump) can distort RT deviations and confound the batch comparison.

## Evidence

- [results] in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large: "in the shorter term the deviations are normally less than 0.05 min, while the differences between different batches can be at least twice as large"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [results] RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches: "RT and m/z deviation data for our long-term QC pool (MetNexs) analyzed within all batches"
- [intro] an open-source Shiny web application and R package: "an open-source Shiny web application and R package"
- [readme] Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
