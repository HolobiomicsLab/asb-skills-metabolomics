---
name: lc-ms-peak-picking-and-grouping
description: Use when starting from raw mzML (or similar vendor) LC–MS files for untargeted
  metabolomics and a feature intensity table (metabolic features × samples) must be
  constructed before statistical analysis, annotation or QC.
when_to_use_negative:
- Input is already a processed feature table or mass_dataset object — peak picking
  would be redundant; downstream cleaning/filtering steps apply instead (e.g. removing
  features with >20% MVs in QC or >50% MVs in study groups).
- A non-R pipeline is mandated and mass_dataset interoperability conversion is not
  desired — massprocesser output is an R mass_dataset object.
- GC–MS or MS/MS-centric preprocessing is the goal and the XCMS-based massprocesser
  parameters do not match the acquisition mode.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: massprocesser
  role: Performs raw data processing, peak picking, grouping and gap filling on the
    mzML directory; generates the mass_dataset-class object
- name: tidyMass
  role: Ecosystem framework that organizes massprocesser/massdataset and provides
    check_tidymass_version()/tidymass_packages() for version verification
  repo: https://github.com/tidymass/tidymass
- name: massdataset
  role: Provides the uniform mass_dataset data structure that stores the feature table
    and processing parameters
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: XCMS
  role: Underlying peak-picking engine on which massprocesser is based
  repo: https://github.com/sneumann/xcms
- name: remotes
  role: Installs massprocesser/tidymass from GitLab or GitHub
  repo: https://gitlab.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:e625c57777fba1dfb4e408c69a3357766e503561091427731f56195aaf29978b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - XCMS
  - massdataset
  - massprocesser
  - remotes
  - tidyMass
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# lc-ms-peak-picking-and-grouping

## Summary

Performs raw LC–MS data processing — peak detection, grouping and gap filling — on deposited mzML files using the massprocesser package (built on XCMS) within the tidyMass ecosystem, producing a mass_dataset-class object that carries both the feature table and the processing parameters.

## When to use

Use when starting from raw mzML (or similar vendor) LC–MS files for untargeted metabolomics and a feature intensity table (metabolic features × samples) must be constructed before statistical analysis, annotation or QC.

## When NOT to use

- Input is already a processed feature table or mass_dataset object — peak picking would be redundant; downstream cleaning/filtering steps apply instead (e.g. removing features with >20% MVs in QC or >50% MVs in study groups).
- A non-R pipeline is mandated and mass_dataset interoperability conversion is not desired — massprocesser output is an R mass_dataset object.
- GC–MS or MS/MS-centric preprocessing is the goal and the XCMS-based massprocesser parameters do not match the acquisition mode.

## Inputs

- Raw LC–MS mzML files (e.g. CRC case-study Supplementary Data 1, also bundled in the jaspershen/tidymass-case-study docker image)
- Peak-picking/grouping processing parameters as reported in the study methods
- R session with the massprocesser and massdataset packages from the tidyMass ecosystem installed via remotes

## Outputs

- mass_dataset-class object storing the feature intensity table and processing parameters
- feature_table.csv with rows = metabolic features and columns = samples
- Peak counts / quality-control metrics comparable to Fig. 3a-b and Supplementary Figs. 5-6

## How to apply

Obtain the raw mzML files (e.g. the CRC case-study files from the tidyMass Supplementary Data 1 or the jaspershen/tidymass-case-study docker image) and install massprocesser from the tidyMass ecosystem via remotes (GitLab/GitHub). Run the massprocesser peak-picking workflow (detect_peaks followed by group_peaks) on the mzML directory using the study's reported processing parameters, so chromatographic peaks are constructed with retention-time grouping and gap filling. Convert the result into the uniform mass_dataset class object, which stores the feature table alongside all processing parameters for traceability. Verify the installed tidyMass ecosystem version with check_tidymass_version()/tidymass_packages() so it matches the published workflow, then export the feature intensity table (rows = metabolic features, columns = samples) as feature_table.csv and compare peak counts and QC metrics against the reported Fig. 3a-b and Supplementary Figs. 5-6. massprocesser is based on XCMS, so parameter semantics follow XCMS peak-picking conventions.

## Related tools

- **massprocesser** (Performs raw data processing, peak picking, grouping and gap filling on the mzML directory; generates the mass_dataset-class object)
- **tidyMass** (Ecosystem framework that organizes massprocesser/massdataset and provides check_tidymass_version()/tidymass_packages() for version verification) — https://github.com/tidymass/tidymass
- **massdataset** (Provides the uniform mass_dataset data structure that stores the feature table and processing parameters) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **XCMS** (Underlying peak-picking engine on which massprocesser is based) — https://github.com/sneumann/xcms
- **remotes** (Installs massprocesser/tidymass from GitLab or GitHub) — https://gitlab.com/tidymass/tidymass

## Examples

```
remotes::install_gitlab("tidymass/massprocesser"); massprocesser::detect_peaks(mzml_dir, ...) %>% group_peaks() # convert to mass_dataset and write feature_table.csv
```

## Evaluation signals

- The result is a mass_dataset-class object (check via class()/massdataset accessors) containing both the feature table and processing parameters.
- Exported feature_table.csv has rows = metabolic features and columns = samples.
- check_tidymass_version()/tidymass_packages() reports the tidyMass ecosystem version matching the published workflow.
- Peak counts and QC metrics are consistent with the paper's Fig. 3a-b and Supplementary Figs. 5-6.
- Peak grouping and gap filling completed without errors across all mzML files in the directory.

## Limitations

- Processing parameters must be taken from the study's reported settings; massprocesser is XCMS-based so parameter semantics follow XCMS, and mismatches change peak counts.
- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, though the mass_dataset class can be converted to other data structures.
- Results should be compared against reported QC figures (Fig. 3a-b, Supplementary Figs. 5-6); deviations indicate parameter or version drift.

## Evidence

- [other] raw data processing, peak picking and grouping are performed by the massProcesser package, which is based on XCMS and generates a “mass_dataset”-class object: [evidence span withheld — anchor status: not_found]
- [abstract] A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [intro] A docker image containing the data, code, and analysis environment is also provided: [evidence span withheld — anchor status: not_found]
- [readme] The xcms R package provides functionality to efficiently preprocess LC-MS (as well as GC-MS and LC-MS/MS) data.: "The *xcms* R package provides functionality to efficiently preprocess LC-MS (as
well as GC-MS and LC-MS/MS) data."
- [intro] Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
