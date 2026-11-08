---
name: s4-class-implementation-in-r
description: Use when building an R data object that must hold a feature × sample
  metabolomics expression matrix together with sample and variable metadata that remain
  dimension-consistent whenever samples or features are removed.
when_to_use_negative:
- You only need a flat feature table export to feed an external tool — tidyMass instead
  converts the mass_dataset class to other data structures for interoperability
- The pipeline target tool is not wrapped or supported by tidyMass, in which case
  the mass_dataset container adds synchronization overhead without benefit
- A one-off script where no object state, metadata pruning, or processing-parameter
  traceability is needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: R
  role: Implementation language for the S4 class (setClass, methods) hosting the mass_dataset
    kernel
- name: tidyMass
  role: Reference object-oriented framework whose uniform mass_dataset data structure
    this class design mirrors
  repo: https://gitlab.com/tidymass/tidymass
- name: tidymass (R package)
  role: Meta-package to install the tidyMass project packages for testing against
    the reference implementation
  repo: https://github.com/tidymass/tidymass
- name: remotes
  role: Used to install tidymass from GitLab/GitHub for environment setup
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:15ca8c3ccfec9683b817e399180840ad35a01cabb62bd5bf7ef739d9c74188d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - remotes
  - tidyMass
  - tidymass (R package)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# s4-class-implementation-in-r

## Summary

Implement an R S4 class (modeled on tidyMass's 'mass_dataset' kernel) that bundles an LC-MS expression matrix with sample_info and variable_info, keeping all metadata slots automatically synchronized under subsetting/filtering and storing processing parameters for traceability.

## When to use

Use when building an R data object that must hold a feature × sample metabolomics expression matrix together with sample and variable metadata that remain dimension-consistent whenever samples or features are removed.

## When NOT to use

- You only need a flat feature table export to feed an external tool — tidyMass instead converts the mass_dataset class to other data structures for interoperability
- The pipeline target tool is not wrapped or supported by tidyMass, in which case the mass_dataset container adds synchronization overhead without benefit
- A one-off script where no object state, metadata pruning, or processing-parameter traceability is needed

## Inputs

- Synthetic or real LC-MS intensity matrix (metabolic features × samples)
- Sample metadata table (sample_info)
- Variable/feature metadata table (variable_info)
- Optional MS2 spectra data and annotation table for the corresponding slots

## Outputs

- Valid S4 'mass_dataset'-style object with six slots (expression_data, sample_info, variable_info, ms2_data, annotation_table, process_info)
- Subsetting/filtering methods that keep all slots dimension-synchronized
- process_info slot recording each processing step and its parameters

## How to apply

Declare the S4 class with setClass, defining the six slots used by tidyMass's mass_dataset kernel: expression_data, sample_info, variable_info, ms2_data, annotation_table, and process_info. Implement the constructor plus subsetting/filtering methods so that sample_info and variable_info rows are automatically pruned and re-ordered to stay aligned with expression_data whenever samples or variables are removed — this synchronization is the core design property of the uniform data format. Write each processing step and its parameters into the process_info slot so prior analysis steps remain traceable, per the tidyMass traceability design. Install the reference implementation for comparison via remotes::install_gitlab("tidymass/tidymass") (or the GitHub mirror). Validate by running validObject() on constructed instances, then exercise a small synthetic LC-MS intensity matrix: remove samples and features and assert that the metadata slots stay dimension-consistent with the expression matrix after each subsetting operation, recording the environment with sessionInfo().

## Related tools

- **R** (Implementation language for the S4 class (setClass, methods) hosting the mass_dataset kernel)
- **tidyMass** (Reference object-oriented framework whose uniform mass_dataset data structure this class design mirrors) — https://gitlab.com/tidymass/tidymass
- **tidymass (R package)** (Meta-package to install the tidyMass project packages for testing against the reference implementation) — https://github.com/tidymass/tidymass
- **remotes** (Used to install tidymass from GitLab/GitHub for environment setup)

## Examples

```
setClass("mass_dataset", slots=list(expression_data="matrix", sample_info="data.frame", variable_info="data.frame", ms2_data="list", annotation_table="data.frame", process_info="list"))
```

## Evaluation signals

- validObject() returns TRUE on the constructed instance (slot types and lengths conform to the class definition)
- After any sample removal, nrow(sample_info) equals ncol(expression_data); after any feature removal, nrow(variable_info) equals nrow(expression_data)
- sample_info and variable_info row order remains aligned with expression_data columns/rows following every subsetting operation
- Every applied processing step and its parameters are retrievable from the process_info slot (traceability check)
- sessionInfo() output captured so the class-implementation environment is reproducible

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; interoperability requires converting the mass_dataset class to other data structures
- The tidymass package is marked with an experimental lifecycle badge in its README
- Installation depends on the remotes package and access to GitLab or GitHub

## Evidence

- [abstract] Uniform data format stores data and parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [intro] mass_dataset is the kernel of tidyMass: [evidence span withheld — anchor status: not_found]
- [intro] Interoperability via mass_dataset conversion: [evidence span withheld — anchor status: not_found]
- [discussion] Some tools not supported in tidyMass: "some widely used metabolomics tools are not wrapped up or supported in tidyMass, it is capable to convert the “mass_dataset” class to more eligible data structures"
- [readme] Install tidymass from GitLab: "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
