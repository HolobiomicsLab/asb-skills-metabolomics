---
name: peak-shape-quality-assessment
description: Use when a targeted MRM workflow (e.g. SmartPeak/OpenMS) has produced
  a FeatureDB CSV of sample types and meta-values such as peak_area, logSN, and RT
  and you need per-compound internal-standard quality scores before quantitative reporting.
when_to_use_negative:
- The input is not a SmartPeak/OpenMS FeatureDB export (e.g. already aggregated peak-area
  tables without transition-level meta-values such as tailing or asymmetry).
- The goal is untargeted feature picking rather than targeted MRM quantitation — SmartPeak
  encapsulates this as an optional step within the targeted metabolomics workflow,
  not for DDA/DIA discovery runs.
- No internal standards are present in the acquisition, so analyte-to-IS height ratios
  cannot be computed.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SmartPeak
  role: Runs the targeted MRM workflow and hosts the optional internal-standard scoring
    step (VALIDATE_IS + COMPUTE_IS_METRICS) as part of its workflow engine
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit whose MRM feature-detection output provides the FeatureDB
    meta-values consumed by the IS scoring step
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: pyOpenMS
  role: Python package for parsing and processing the workflow's exported feature
    files when building the per-compound IS score table
- name: SmartPeak CLI
  role: Command-line equivalent of the SmartPeak GUI for running the targeted quantitation
    workflow that includes the IS scoring step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:31369b70dc1cf443131e83675afe0ac57c03bf8cb99a69c3892b9058f418e93e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# peak-shape-quality-assessment

## Summary

Optional internal-standard scoring module in the SmartPeak targeted-quantitation workflow that computes per-compound IS metrics (peak tailing, asymmetry, analyte-to-IS height ratio) from a FeatureDB export and emits a per-compound score table (is_scores.csv).

## When to use

Use when a targeted MRM workflow (e.g. SmartPeak/OpenMS) has produced a FeatureDB CSV of sample types and meta-values such as peak_area, logSN, and RT and you need per-compound internal-standard quality scores before quantitative reporting.

## When NOT to use

- The input is not a SmartPeak/OpenMS FeatureDB export (e.g. already aggregated peak-area tables without transition-level meta-values such as tailing or asymmetry).
- The goal is untargeted feature picking rather than targeted MRM quantitation — SmartPeak encapsulates this as an optional step within the targeted metabolomics workflow, not for DDA/DIA discovery runs.
- No internal standards are present in the acquisition, so analyte-to-IS height ratios cannot be computed.

## Inputs

- FeatureDB CSV exported from the SmartPeak/OpenMS targeted MRM workflow (one row per feature, with sample type and meta-values such as peak_area, logSN, RT)
- Transition/component group table mapping each analyte component to its designated internal standard
- Acquisition meta-values for peak heights, tailing, and asymmetry stored on the MRM transitions

## Outputs

- is_scores.csv — tidy table with one row per compound, columns for the aggregated IS score and the three components (tailing factor, asymmetry factor, analyte-to-IS height ratio)
- Validation flags for compounds failing the VALIDATE_IS checks (missing IS pairing, null or out-of-range meta-values)

## How to apply

Load the FeatureDB CSV exported from the targeted MRM workflow (e.g. FeatureDB.csv carrying sample types plus meta-values such as peak_area, logSN, RT) into a pandas DataFrame. First run the VALIDATE_IS step: confirm every analyte component is paired with its designated internal standard and that the required acquisition meta-values (peak heights, tailing, asymmetry) are present and well-formed, flagging compounds with missing or out-of-range IS meta-values rather than silently scoring them. Then run COMPUTE_IS_METRICS: for each compound/injection calculate the three IS score components — peak tailing factor, peak asymmetry factor, and analyte-to-IS height ratio — from the stored transition meta-values. Aggregate the three components into a single per-compound IS score and assemble a tidy table with one row per compound and columns for the score and each component, then write it to is_scores.csv. Rationale: SmartPeak is built on the OpenMS toolkit and the workflow is initialized by validating the acquisition meta-values of transitions, components, and component groups, so the IS scoring step fits the same validate-then-compute pattern used by the enclosing PICK_MRM_FEATURES → SELECT_FEATURES pipeline.

## Related tools

- **SmartPeak** (Runs the targeted MRM workflow and hosts the optional internal-standard scoring step (VALIDATE_IS + COMPUTE_IS_METRICS) as part of its workflow engine) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit whose MRM feature-detection output provides the FeatureDB meta-values consumed by the IS scoring step) — https://github.com/AutoFlowResearch/SmartPeak
- **pyOpenMS** (Python package for parsing and processing the workflow's exported feature files when building the per-compound IS score table)
- **SmartPeak CLI** (Command-line equivalent of the SmartPeak GUI for running the targeted quantitation workflow that includes the IS scoring step)

## Examples

```
from pyopenms import *; featuredb = FeatureDB('FeatureDB.csv'); is_scores = featuredb.getIsScores(compounds)  # write is_scores.csv with tailing, asymmetry, height-ratio columns
```

## Evaluation signals

- Every analyte row in the output table has a designated IS pairing (no unmatched analyte components).
- All three component columns (tailing factor, asymmetry factor, analyte-to-IS height ratio) are non-null for every scored compound.
- Compounds that failed VALIDATE_IS (missing or out-of-range meta-values) are flagged rather than silently scored.
- Output file is_scores.csv exists with one row per compound and both the aggregated score and the three component columns.
- The computed height-ratio values are consistent with the analyte and IS peak-height meta-values stored in the input FeatureDB.

## Limitations

- The IS scoring step is optional and only applies to targeted MRM quantitation; SmartPeak's other workflows (e.g. FIAMS FullScan Unknowns, LC-MS/MS-DDA, Non-targeted LC-MS/MS DIA analysis) do not include it.
- Scoring depends on the enclosing workflow having correctly validated acquisition meta-values of transitions, components, and component groups beforehand; missing or malformed meta-values cannot be scored.
- GUI feature-table viewing is capped at 5000 entries and chromatogram viewing at 9000 points, which constrains interactive review of large sequences during IS-score inspection.
- SmartPeak supports most workflow steps via OpenMS, but aggregating standards, finding peaks, aligning, applying quality control, and writing to file are implemented in SmartPeak itself, so the step's availability depends on the installed version.

## Evidence

- [methods] Optional internal-standard scoring step computes per-compound IS metrics from the FeatureDB produced by the MRM workflow: [evidence span withheld — anchor status: not_found]
- [methods] The workflow validates the acquisition meta-values of the MRM transitions before scoring: [evidence span withheld — anchor status: not_found]
- [results] The enclosing workflow covers the targeted metabolomics steps including peak picking and quantitation: [evidence span withheld — anchor status: not_found]
