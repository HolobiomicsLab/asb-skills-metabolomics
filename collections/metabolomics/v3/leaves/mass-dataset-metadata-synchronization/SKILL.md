---
name: mass-dataset-metadata-synchronization
description: Use when building or modifying an R metabolomics data container that
  must keep sample metadata and variable metadata rows aligned with a feature (metabolic
  feature) intensity matrix after subsetting or filtering operations.
when_to_use_negative:
- Input is already a finalized feature table for a tool that needs its own format—instead
  convert the mass_dataset to the other tool's data structure via the interoperability
  pathway
- A non-object-oriented pipeline (e.g. plain MZmine, MS-DIAL, XCMS peak tables) where
  no metadata synchronization container is required
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: tidyMass
  role: Reference implementation of the mass_dataset uniform data structure and its
    synchronization semantics
  repo: https://www.tidymass.org/
- name: massdataset
  role: Package providing the mass_dataset class and conversion/interoperability with
    other tools' data structures
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: R
  role: Implementation language for the S4 class and subsetting methods
- name: remotes
  role: Used to install tidymass from GitLab
  repo: https://gitlab.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:bcfe84f033a16614fb8c97faa7ac27a470855f4371c4ab38a6558914cf2147cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - massdataset
  - remotes
  - tidyMass
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# mass-dataset-metadata-synchronization

## Summary

Reconstructs tidyMass's 'mass_dataset' uniform data structure—the kernel of tidyMass—whose expression_data, sample_info, variable_info, ms2_data, annotation_table, and process_info slots stay automatically synchronized when samples or features are subset, and whose process_info slot records parameters for traceability.

## When to use

Use when building or modifying an R metabolomics data container that must keep sample metadata and variable metadata rows aligned with a feature (metabolic feature) intensity matrix after subsetting or filtering operations.

## When NOT to use

- Input is already a finalized feature table for a tool that needs its own format—instead convert the mass_dataset to the other tool's data structure via the interoperability pathway
- A non-object-oriented pipeline (e.g. plain MZmine, MS-DIAL, XCMS peak tables) where no metadata synchronization container is required

## Inputs

- LC-MS metabolic feature intensity matrix (features x samples)
- sample_info data frame (sample metadata, one row per sample)
- variable_info data frame (variable/feature metadata, one row per feature)
- optional MS2 spectra (ms2_data) and annotation_table

## Outputs

- mass_dataset-class S4 object with dimension-synchronized expression_data, sample_info, and variable_info slots
- process_info slot containing the recorded processing steps and parameters
- validObject()-validated instance with consistent slot dimensions

## How to apply

Define the S4 class with setClass, declaring the slots the paper describes (expression_data, sample_info, variable_info, plus ms2_data, annotation_table, process_info per the task card). Implement constructor and subsetting/filtering methods so that whenever samples or metabolic features are removed from expression_data, the corresponding sample_info and variable_info rows are automatically pruned and re-ordered to remain dimension-consistent—this is the core 'automatic component synchronization' property of the tidyMass uniform data format. Record each processing step and its parameters in the process_info slot so prior analysis steps and parameters remain traceable, as the paper's uniform data format was 'specifically designed to store and manage processed metabolomics data and processing parameters'. Exercise the semantics on a small synthetic LC-MS intensity matrix by removing samples and features, then verify with validObject() and dimension checks after each subsetting operation; record the environment with sessionInfo(). Install the reference implementation via remotes::install_gitlab("tidymass/tidymass") as prescribed in the methods.

## Related tools

- **tidyMass** (Reference implementation of the mass_dataset uniform data structure and its synchronization semantics) — https://www.tidymass.org/
- **massdataset** (Package providing the mass_dataset class and conversion/interoperability with other tools' data structures) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **R** (Implementation language for the S4 class and subsetting methods)
- **remotes** (Used to install tidymass from GitLab) — https://gitlab.com/tidymass/tidymass

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- validObject() returns TRUE on the constructed mass_dataset instance
- After any sample removal, nrow(sample_info) equals ncol(expression_data) and row order matches column names
- After any feature removal, nrow(variable_info) equals nrow(expression_data) and row order matches rownames
- process_info slot contains an entry with parameters for every processing step applied, enabling tracing of prior analysis steps
- sessionInfo() output recorded alongside the object for environment reproducibility

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, so the mass_dataset must be converted to other data structures for interoperability
- tidymass package is marked lifecycle-experimental in its README, so the class API may change

## Evidence

- [abstract] uniform data format designed to store and manage processed metabolomics data and processing parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [intro] mass_dataset kernel: [evidence span withheld — anchor status: not_found]
- [intro] interoperability: [evidence span withheld — anchor status: not_found]
- [methods] install from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [readme] install command: "remotes::install_gitlab("tidymass/tidymass")"
- [discussion] missing tools: "some widely used metabolomics tools are not wrapped up or supported in tidyMass, it is capable to convert the “mass_dataset” class to more eligible data structures for these tools in the future."
