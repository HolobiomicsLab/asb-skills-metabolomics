---
name: mass-dataset-object-construction
description: Use when LC–MS untargeted metabolomics raw data (e.g. mzML files) have
  been peak-picked/grouped with massprocesser or a cleaned feature table exists, and
  downstream tidyMass steps (massCleaner, massstat, metpath, metid) require the unified
  mass_dataset object.
when_to_use_negative:
- Input is already a non-tidyMass-format feature table that only needs conversion
  for another tool — use the interoperability conversion path instead of re-processing
- The analysis target is raw spectra inspection or instrument-side QC rather than
  building the uniform tidyMass data object for downstream cleaning/statistics
- The workflow requires tools that are not wrapped in tidyMass and cannot accept the
  mass_dataset class without conversion
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: tidymass
  role: Umbrella R package organizing all tidyMass-project packages; installed via
    remotes before constructing the object
  repo: https://github.com/tidymass/tidymass
- name: massprocesser
  role: Performs XCMS-based raw data processing, peak picking and grouping, producing
    the result that is converted into the mass_dataset object
- name: massdataset
  role: Provides the mass_dataset class and its construction/manipulation and interoperability
    conversion functions
- name: XCMS
  role: Underlying peak-picking engine on which massprocesser's raw-data processing
    is based
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:0e661f873b72400c17f6f55ad0e826e23b8343f1b6855fd4151a74ea4353c278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - XCMS
  - dplyr
  - ggplot2
  - masscleaner
  - massdataset
  - massprocesser
  - massstat
  - metpath
  - purrr
  - remotes
  - tidymass
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# mass-dataset-object-construction

## Summary

Constructs the uniform 'mass_dataset'-class object that is the kernel of the tidyMass ecosystem, storing the processed feature intensity table (rows = metabolic features, columns = samples) together with sample metadata, variable metadata, and the processing parameters so prior analysis steps remain traceable.

## When to use

Use when LC–MS untargeted metabolomics raw data (e.g. mzML files) have been peak-picked/grouped with massprocesser or a cleaned feature table exists, and downstream tidyMass steps (massCleaner, massstat, metpath, metid) require the unified mass_dataset object.

## When NOT to use

- Input is already a non-tidyMass-format feature table that only needs conversion for another tool — use the interoperability conversion path instead of re-processing
- The analysis target is raw spectra inspection or instrument-side QC rather than building the uniform tidyMass data object for downstream cleaning/statistics
- The workflow requires tools that are not wrapped in tidyMass and cannot accept the mass_dataset class without conversion

## Inputs

- Raw LC–MS mzML files (e.g. from the CRC case study, Supplementary Data 1 or the jaspershen/tidymass-case-study docker image)
- Peak-picking/grouping result from massprocesser (XCMS-based detect_peaks/group_peaks)
- Cleaned feature intensity table with features as rows and samples as columns (post massCleaner filtering)
- Sample metadata (sample_info) and feature/variable metadata (variable_info)

## Outputs

- mass_dataset-class object holding the feature table, sample info, variable info, and processing parameters
- Exported feature_table.csv (features × samples intensity matrix)

## How to apply

First install the tidyMass ecosystem from GitLab or GitHub via remotes, then run the massprocesser peak-picking workflow (detect_peaks/group_peaks) on the mzML file directory with the study's reported parameters for retention-time grouping and gap filling. Convert the processing result into a mass_dataset object, which stores both the feature table and the processing parameters used to generate it. Populate the object so that the expression/feature intensity matrix has metabolic features as rows and samples as columns, alongside sample information and variable (feature) information slots. Verify the tidyMass ecosystem version with check_tidymass_version()/tidymass_packages() to confirm it matches the published workflow, then persist the feature table (e.g. feature_table.csv) and, if needed for external tools, convert the mass_dataset class to other data structures via the documented interoperability pathway.

## Related tools

- **tidymass** (Umbrella R package organizing all tidyMass-project packages; installed via remotes before constructing the object) — https://github.com/tidymass/tidymass
- **massprocesser** (Performs XCMS-based raw data processing, peak picking and grouping, producing the result that is converted into the mass_dataset object)
- **massdataset** (Provides the mass_dataset class and its construction/manipulation and interoperability conversion functions)
- **XCMS** (Underlying peak-picking engine on which massprocesser's raw-data processing is based) — https://github.com/sneumann/xcms

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- The resulting object is of class mass_dataset and its expression slot is a feature intensity matrix with rows = metabolic features and columns = samples
- check_tidymass_version()/tidymass_packages() report a tidyMass ecosystem version consistent with the published workflow (Nat Commun 13, 4365, 2022)
- The object's stored processing parameters reproduce the raw-data processing steps actually applied (peak detection, retention-time grouping, gap filling)
- Exported feature_table.csv feature/sample counts and QC metrics are comparable to the paper's reported Fig. 3a-b and Supplementary Figs. 5-6
- Conversion of the mass_dataset to an external tool's data structure round-trips without loss of the feature table

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; the mass_dataset class must be converted to other data structures for those tools
- The mass_dataset object captures tidyMass-specific processing parameters; analyses outside the tidyMass pipeline may not consume it directly

## Evidence

- [intro] A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [intro] tidyMass has high interoperability with other existing tools, converting "mass_dataset" class with other data structures required by other tools: [evidence span withheld — anchor status: not_found]
- [readme] `tidymass` is a R package which is used to manage and organize all the pacakges in [tidyMass project](https://www.tidymass.org/).: [evidence span withheld — anchor status: not_found]
