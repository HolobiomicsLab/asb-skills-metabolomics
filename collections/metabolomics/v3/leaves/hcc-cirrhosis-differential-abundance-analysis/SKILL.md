---
name: hcc-cirrhosis-differential-abundance-analysis
description: Use when a final high-quality peak table from WiPP's result integration
  step on the HCC vs cirrhosis plasma GC-MS dataset is available and the goal is to
  identify features significantly different between the two conditions.
when_to_use_negative:
- Input is already a processed feature table unrelated to the WiPP HCC vs cirrhosis
  dataset
- The WiPP peak detection and result integration pipeline has not yet been run, so
  no high-quality peak table exists
- The goal is peak detection quality assessment rather than differential abundance
  between conditions
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: WiPP
  role: Produces the final high-quality peak table (classifier output merged across
    peak detection algorithms) on which the HCC vs cirrhosis differential comparison
    is performed
  repo: https://github.com/bihealth/WiPP
- name: Python 3
  role: Language used to load the peak table, filter annotated features, compute/extract
    fold changes and p-values, and write the structured feature record
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo9090171
    title: 'WiPP: Workflow for Improved Peak Picking for Gas Chromatography-Mass Spectrometry
      (GC-MS) Data'
  source_content_hash: sha256:8ceba4e1041fb1391653e48ec0f250edc69193c424b2e5727a5cc9f8e4e360ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bihealth/WiPP
  tools:
  - Python 3
  - WiPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hcc-cirrhosis-differential-abundance-analysis

## Summary

Statistical comparison of annotated GC-MS metabolite features between HCC and cirrhosis plasma samples, using fold change and p-values computed on the high-quality peak table produced by WiPP's result integration step.

## When to use

Use when a final high-quality peak table from WiPP's result integration step on the HCC vs cirrhosis plasma GC-MS dataset is available and the goal is to identify features significantly different between the two conditions.

## When NOT to use

- Input is already a processed feature table unrelated to the WiPP HCC vs cirrhosis dataset
- The WiPP peak detection and result integration pipeline has not yet been run, so no high-quality peak table exists
- The goal is peak detection quality assessment rather than differential abundance between conditions

## Inputs

- Final high-quality peak table from WiPP result integration (peak classification output merged across peak detection algorithms)
- Feature annotations from the WiPP annotation stage (retention index supported)
- Wash/blank sample files containing alkanes for retention index calculation
- Original published analysis results (Table 1 of Ranjbar et al.) for cross-reference

## Outputs

- Fold change and p-value per feature for HCC vs cirrhosis comparison
- Structured feature record (e.g., hexose_feature_record.csv) with annotation, fold change, and p-value
- List of WiPP-detected significantly different features not reported in the original study

## How to apply

Load the final high-quality peak table produced by WiPP's result integration step, where classifier quality information is merged with results from different peak detection algorithms. Use the annotation assigned during the WiPP annotation stage (retention-index-supported where alkanes in wash samples are available) to filter the peak table for features annotated as hexose or other target analytes. Extract fold change (HCC vs cirrhosis) and p-value from the statistical comparison of the two experimental condition groups. Cross-reference findings against Table 1 of the original Ranjbar et al. study to distinguish WiPP-specific discoveries from previously reported metabolites. Write each validated feature to a structured record such as hexose_feature_record.csv for expert review.

## Related tools

- **WiPP** (Produces the final high-quality peak table (classifier output merged across peak detection algorithms) on which the HCC vs cirrhosis differential comparison is performed) — https://github.com/bihealth/WiPP
- **Python 3** (Language used to load the peak table, filter annotated features, compute/extract fold changes and p-values, and write the structured feature record)

## Examples

```
cd ./projects/example_project && ../../run_WiPP.sh pp -n 4 && python -c "import pandas as pd; df = pd.read_csv('hexose_feature_record.csv'); print(df)"
```

## Evaluation signals

- The hexose feature record includes annotation, fold change, and p-value and is confirmed absent from Table 1 of the original Ranjbar et al. study
- Six of nine validated analytes from the original study are confirmed with comparable fold changes and p-values
- Peak table used is the merged output across peak detection algorithms (result integration step), not a single-algorithm table
- Expert review confirms annotation, fold change, and p-value against the original published analysis
- Features compared are drawn from the HCC vs cirrhosis two-condition grouping defined in the WiPP Input_folder structure

## Limitations

- The WiPP example classifier from the repository is specific to the test data and must not be used for real project analysis; a project-specific trained SVM classifier (typically ~1200 peaks per algorithm annotated once per instrument/protocol) is required for reliable peak quality classification
- WiPP has been tested on Ubuntu 16 and CentOS 7 only; Ubuntu 18 is unsupported due to lacking support of incorporated R packages
- Retention index calculation requires optional wash/blank samples containing alkanes (c10 through c34 list); annotation support is weaker without them

## Evidence

- [abstract] The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set: "The quality information returned by the classifier for each individual peak is merged with results from different peak detection algorithms to create one final high-quality peak set"
- [results] Six of nine validated analytes confirmed: [evidence span withheld — anchor status: not_found]
- [readme] Peak annotation effort per algorithm: "Peak annotation usually takes several hours (1200 peaks per algorithm by default), but you only need to do that once per instrument/protocol"
- [readme] Retention index support: "Here, we assume that these samples contain the alkanes that are used for Retention Index calculation"
