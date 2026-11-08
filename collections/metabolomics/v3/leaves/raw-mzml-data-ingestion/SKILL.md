---
name: raw-mzml-data-ingestion
description: Use when starting from raw LC-MS mzML files and an untargeted metabolomics
  feature table (rows = metabolic features, columns = samples) must be generated before
  any statistics or annotation, following the tidyMass CRC case-study workflow.
when_to_use_negative:
- Input is already a processed feature table (e.g. CSV/Excel peak table) rather than
  raw mzML files — load it directly into a mass_dataset via massdataset instead of
  re-running peak picking
- The pipeline requires tools not wrapped by tidyMass, since some widely used metabolomics
  tools are not supported — though mass_dataset can be converted to other tools' data
  structures
- GC-MS or non-LC-MS data, since the skill targets LC-MS-based untargeted metabolomics
  workflows
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: massprocesser
  role: Performs raw data processing, peak picking and grouping on mzML files and
    generates the mass_dataset-class object
  repo: https://gitlab.com/tidymass/tidymass
- name: massdataset
  role: Provides the uniform mass_dataset data structure storing the feature table
    and processing parameters
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Umbrella R package to install/organize the tidyMass ecosystem packages and
    verify versions
  repo: https://github.com/tidymass/tidymass
- name: XCMS
  role: Underlying peak-picking engine on which massprocesser is based
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:51630a0503d7013b30438467e288e25d6b22ff7914b98b05f6f9c887f93ed170
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - XCMS
  - massdataset
  - massprocesser
  - remotes
  - tidymass
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# raw-mzml-data-ingestion

## Summary

Ingest raw LC-MS mzML files and run peak detection, grouping and gap filling via tidyMass's massprocesser (built on XCMS), producing a mass_dataset-class object that stores both the feature intensity table and processing parameters for downstream tidyMass analysis.

## When to use

Use when starting from raw LC-MS mzML files and an untargeted metabolomics feature table (rows = metabolic features, columns = samples) must be generated before any statistics or annotation, following the tidyMass CRC case-study workflow.

## When NOT to use

- Input is already a processed feature table (e.g. CSV/Excel peak table) rather than raw mzML files — load it directly into a mass_dataset via massdataset instead of re-running peak picking
- The pipeline requires tools not wrapped by tidyMass, since some widely used metabolomics tools are not supported — though mass_dataset can be converted to other tools' data structures
- GC-MS or non-LC-MS data, since the skill targets LC-MS-based untargeted metabolomics workflows

## Inputs

- Raw mzML files of the LC-MS untargeted metabolomics run (e.g. CRC case-study mzML files from Supplementary Data 1)
- Directory path containing the mzML files
- Peak-detection, grouping and gap-filling parameters as reported in the paper

## Outputs

- mass_dataset-class object containing the feature intensity table (rows = metabolic features, columns = samples) plus processing parameters
- feature_table.csv exported from the mass_dataset object
- Peak counts and QC metrics for comparison with Fig. 3a-b and Supplementary Figs. 5-6

## How to apply

Obtain the raw mzML files of the CRC case study from the tidyMass case-study resources (Supplementary Data 1, also bundled in the jaspershen/tidymass-case-study docker image). Install massprocesser from the tidyMass ecosystem via remotes (remotes::install_gitlab("tidymass/tidymass") pulls the umbrella package). Invoke the massprocesser peak-picking workflow (detect_peaks / group_peaks) on the mzML file directory using the paper's reported processing parameters so peaks are constructed with retention-time grouping and gap filling; massprocesser is based on XCMS. Convert the result into the uniform mass_dataset class object that stores both the feature table and the processing parameters, enabling tracing of prior analysis steps. Confirm the tidyMass ecosystem version with check_tidymass_version()/tidymass_packages(), then export the feature intensity table to feature_table.csv and compare peak counts and QC metrics against reported Fig. 3a-b and Supplementary Figs. 5-6.

## Related tools

- **massprocesser** (Performs raw data processing, peak picking and grouping on mzML files and generates the mass_dataset-class object) — https://gitlab.com/tidymass/tidymass
- **massdataset** (Provides the uniform mass_dataset data structure storing the feature table and processing parameters) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Umbrella R package to install/organize the tidyMass ecosystem packages and verify versions) — https://github.com/tidymass/tidymass
- **XCMS** (Underlying peak-picking engine on which massprocesser is based) — https://github.com/sneumann/xcms

## Examples

```
remotes::install_gitlab("tidymass/tidymass"); object <- massprocesser::detect_peaks(mzml_dir, polarity = "positive")
```

## Evaluation signals

- The output is a mass_dataset-class object (verifiable with class(obj)) that stores both the feature table and the processing parameters
- Peak counts and QC metrics match the reported Fig. 3a-b and Supplementary Figs. 5-6 within tolerance
- check_tidymass_version()/tidymass_packages() confirms the tidyMass ecosystem version matches the published workflow
- Exported feature_table.csv has rows = metabolic features and columns = samples
- The processing history recorded in the mass_dataset object allows tracing of the prior analysis steps and parameters

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, although the mass_dataset class can be converted to more eligible data structures for those tools
- Reproducibility depends on matching the paper's reported processing parameters and the correct tidyMass ecosystem version; wrong versions or parameters will shift peak counts
- Processing requires optimization of multiple steps (raw data processing, data cleaning, QC), so defaults may need tuning per dataset

## Evidence

- [other] raw data processing, peak picking and grouping are performed by the massProcesser package, which is based on XCMS and generates a “mass_dataset”-class object: [evidence span withheld — anchor status: not_found]
- [abstract] A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [methods] You can install `tidymass` from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [intro] A docker image containing the data, code, and analysis environment is also provided: [evidence span withheld — anchor status: not_found]
- [intro] Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
