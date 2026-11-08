---
name: internal-standard-scoring
description: Use when processing targeted LC-MS/MS MRM quantitation data (e.g. SmartPeak
  FeatureDB.csv from the 5500 QTRAP RapidRIP tutorial) and you want a per-compound
  IS quality score computed from tailing, asymmetry, and height-ratio metrics before
  quantitation reporting.
when_to_use_negative:
- Input is already a quantitation report or a feature table that lacks raw IS peak-height/tailing/asymmetry
  meta-values — VALIDATE_IS will simply flag everything and no scores can be computed.
- The experiment is untargeted FIA-MS or DDA/DIA (e.g. the Thermo Orbitrap tutorials)
  where no IS pairing or transition-level meta-values exist.
- You only need final concentrations from the calibration curve — IS scoring is an
  optional pre-quantitation QC step, not a quantitation method.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SmartPeak
  role: Emits the FeatureDB and implements the optional internal standard scoring
    step (VALIDATE_IS, COMPUTE_IS_METRICS) in the targeted MRM workflow
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: Underlying toolkit whose MRMFeatureScoring/QC meta-values (tailing, asymmetry,
    height ratios) the scoring step consumes; current OpenMS versions support most
    steps except Aggregating standards, Find peaks, Align, Apply quality control,
    Write to file, which SmartPeak adds
  repo: https://github.com/OpenMS/OpenMS
- name: pyOpenMS
  role: Used to parse and process the FeatureDB / mzTab-style outputs the scoring
    step consumes
  repo: https://pyopenms.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:0627522cd0e5ac53fec20f4e2aec388f79e4488582a94418c627a584c21f8d87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - OpenMS
  - SmartPeak
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Internal standard scoring

## Summary

An optional SmartPeak workflow step that computes per-compound internal-standard quality metrics (peak tailing factor, peak asymmetry factor, and analyte-to-IS height ratio) from the FeatureDB produced by the targeted MRM workflow and emits a per-compound score table.

## When to use

Use when processing targeted LC-MS/MS MRM quantitation data (e.g. SmartPeak FeatureDB.csv from the 5500 QTRAP RapidRIP tutorial) and you want a per-compound IS quality score computed from tailing, asymmetry, and height-ratio metrics before quantitation reporting.

## When NOT to use

- Input is already a quantitation report or a feature table that lacks raw IS peak-height/tailing/asymmetry meta-values — VALIDATE_IS will simply flag everything and no scores can be computed.
- The experiment is untargeted FIA-MS or DDA/DIA (e.g. the Thermo Orbitrap tutorials) where no IS pairing or transition-level meta-values exist.
- You only need final concentrations from the calibration curve — IS scoring is an optional pre-quantitation QC step, not a quantitation method.

## Inputs

- FeatureDB.csv exported from the SmartPeak targeted MRM workflow (with sample types and meta-values such as peak_area, logSN, RT)
- Transition list mapping each analyte to its designated internal standard
- Acquisition meta-values for peak heights, tailing, and asymmetry per transition

## Outputs

- is_scores.csv: tidy table with one row per compound, columns for the single IS score and its tailing, asymmetry, and height-ratio components
- Flags/validation report for compounds that fail VALIDATE_IS checks (missing IS pairing or null/out-of-range meta-values)

## How to apply

Load the FeatureDB CSV exported from the SmartPeak MRM workflow — the table that carries sample types and meta-values such as peak_area, logSN, and RT — into a pandas DataFrame, since SmartPeak writes it via the pyOpenMS-compatible format. Run the VALIDATE_IS step first: check that every analyte transition is paired with its designated internal standard and that required acquisition meta-values (peak heights, tailing, asymmetry) are present and well-formed, flagging rather than silently scoring compounds that fail. Then run COMPUTE_IS_METRICS: for each compound/injection calculate the peak tailing factor, peak asymmetry factor, and analyte-to-IS height ratio, aggregate the three components into a single per-compound IS score, and assemble a tidy per-compound table. Write the result to is_scores.csv with one row per compound and columns for the score and each component. The rationale is that SmartPeak 'encapsulates advanced algorithms' for targeted metabolomics, and scoring the IS quality before quantitation catches integrations whose IS peaks are distorted (tailing/asymmetry) or whose analyte/IS response deviates, enabling later flagging rather than false quantitation.

## Related tools

- **SmartPeak** (Emits the FeatureDB and implements the optional internal standard scoring step (VALIDATE_IS, COMPUTE_IS_METRICS) in the targeted MRM workflow) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (Underlying toolkit whose MRMFeatureScoring/QC meta-values (tailing, asymmetry, height ratios) the scoring step consumes; current OpenMS versions support most steps except Aggregating standards, Find peaks, Align, Apply quality control, Write to file, which SmartPeak adds) — https://github.com/OpenMS/OpenMS
- **pyOpenMS** (Used to parse and process the FeatureDB / mzTab-style outputs the scoring step consumes) — https://pyopenms.readthedocs.io/en/latest/

## Examples

```
python -c "import pandas as pd; db=pd.read_csv('FeatureDB.csv'); print(db[['peak_area','logSN','RT']].isna().sum())"  # then run SmartPeak workflow with VALIDATE_IS, COMPUTE_IS_METRICS → is_scores.csv
```

## Evaluation signals

- Every analyte row in is_scores.csv has a non-null value for all three components (tailing, asymmetry, height ratio) — nulls indicate a VALIDATE_IS failure that was not flagged.
- The number of compounds written to is_scores.csv equals the number of analyte/IS pairs in the transition list minus compounds explicitly flagged by VALIDATE_IS.
- Tailing and asymmetry component values stay in the expected chromatographic range (near 1 for symmetric peaks); grossly out-of-range values should trace back to a flagged or distorted IS peak.
- Height-ratio component values are consistent with the analyte and IS peak-height meta-values loaded from FeatureDB (recomputable from those columns).
- The score column is a deterministic aggregation of the three components — re-running the step on the same FeatureDB yields identical scores.

## Limitations

- The step is optional and only meaningful for targeted MRM experiments where each analyte has a designated internal standard; untargeted or FIA-MS data provide no IS pairing.
- Compounds whose IS peaks are missing, distorted, or lack required meta-values are flagged rather than scored — the workflow does not impute or recover them.
- The scoring depends on upstream peak picking quality: if MRM feature detection or background-interference estimation was wrong, the tailing/asymmetry/height-ratio metrics inherit those errors.
- Only the three IS components (tailing, asymmetry, height ratio) are computed; the workflow does not cover absolute concentration QC, which is handled by separate feature-QC steps.

## Evidence

- [methods] Optional IS scoring step that validates IS pairing and computes tailing, asymmetry, and height-ratio components, emitting is_scores.csv: [evidence span withheld — anchor status: not_found]
- [methods] VALIDATE_IS checks IS pairing before scoring; COMPUTE_IS_METRICS computes the three components: [evidence span withheld — anchor status: not_found]
- [results] SmartPeak supports all required targeted metabolomics steps including internal-standard QC and reporting: [evidence span withheld — anchor status: not_found]
