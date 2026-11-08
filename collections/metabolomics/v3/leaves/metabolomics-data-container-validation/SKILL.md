---
name: metabolomics-data-container-validation
description: Use when a processed LC-MS feature table must be stored in a uniform,
  traceable object and you need sample/variable metadata to stay dimension-consistent
  with the intensity matrix after subsetting, filtering, or removing samples or features.
when_to_use_negative:
- The input is raw vendor LC-MS data needing peak picking/grouping first — that preprocessing
  is done by tools like MZmine, MS-DIAL, XCMS, OpenMS, or tidyMass's own raw-data
  modules before a feature table exists to wrap.
- The output must feed a tool that cannot consume mass_dataset objects — tidyMass
  instead converts the mass_dataset class to the other tool's required data structure
  (interoperability route), and some widely used metabolomics tools are still not
  wrapped up or supported in tidyMass.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: tidyMass
  role: Supplies the mass_dataset uniform data structure and the object-oriented framework
    whose kernel this skill reconstructs and validates
  repo: https://github.com/tidymass/tidymass
- name: massdataset
  role: Provides the mass_dataset class and its interoperability conversions with
    other tools' data structures
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: remotes
  role: Used to install tidymass from GitLab (remotes::install_gitlab("tidymass/tidymass"))
    to set up the execution environment
- name: R
  role: Language in which the S4 class (setClass), synchronization methods, validObject()
    checks, and sessionInfo() recording are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:972f7fede26dfe5678528ef920f4c102ae7890931d070dcbaef43ee1c1c4763b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - R
  - massdataset
  - remotes
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-data-container-validation

## Summary

Wraps processed LC-MS metabolomics data in a uniform container object (tidyMass's 'mass_dataset' kernel) whose expression matrix, sample_info, variable_info, ms2_data, annotation_table, and process_info slots stay automatically synchronized, then validates the container's structural integrity.

## When to use

Use when a processed LC-MS feature table must be stored in a uniform, traceable object and you need sample/variable metadata to stay dimension-consistent with the intensity matrix after subsetting, filtering, or removing samples or features.

## When NOT to use

- The input is raw vendor LC-MS data needing peak picking/grouping first — that preprocessing is done by tools like MZmine, MS-DIAL, XCMS, OpenMS, or tidyMass's own raw-data modules before a feature table exists to wrap.
- The output must feed a tool that cannot consume mass_dataset objects — tidyMass instead converts the mass_dataset class to the other tool's required data structure (interoperability route), and some widely used metabolomics tools are still not wrapped up or supported in tidyMass.

## Inputs

- LC-MS intensity matrix (metabolic feature table; rows = features, columns = samples)
- sample metadata table (sample_info)
- variable metadata table (variable_info)
- MS2 spectra data (ms2_data)
- annotation table
- processing parameters recorded per workflow step

## Outputs

- Validated mass_dataset object with dimension-consistent expression_data, sample_info, and variable_info slots
- validObject() pass/fail report for the constructed instance
- process_info slot containing the parameters of each prior processing step
- sessionInfo() record of the execution environment

## How to apply

Define the container class with setClass in R, declaring the six slots that constitute the mass_dataset kernel: expression_data, sample_info, variable_info, ms2_data, annotation_table, and process_info. Implement the constructor and subsetting/filtering methods so that sample_info and variable_info rows are automatically pruned and re-ordered to remain aligned with expression_data whenever samples or variables are removed. Record each processing step and its parameters in the process_info slot, which is what makes prior analysis steps traceable, and use validObject() to check the constructed instance. Exercise the semantics on a small synthetic LC-MS intensity matrix by removing samples and features and asserting that the metadata slots stay dimension-consistent with the expression matrix. Install the tidyMass ecosystem with remotes::install_gitlab("tidymass/tidymass") (or the GitHub mirror), and record the environment with sessionInfo() for reproducibility.

## Related tools

- **tidyMass** (Supplies the mass_dataset uniform data structure and the object-oriented framework whose kernel this skill reconstructs and validates) — https://github.com/tidymass/tidymass
- **massdataset** (Provides the mass_dataset class and its interoperability conversions with other tools' data structures) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **remotes** (Used to install tidymass from GitLab (remotes::install_gitlab("tidymass/tidymass")) to set up the execution environment)
- **R** (Language in which the S4 class (setClass), synchronization methods, validObject() checks, and sessionInfo() recording are implemented)

## Examples

```
Rscript -e 'remotes::install_gitlab("tidymass/tidymass"); od <- new("mass_dataset", expression_data = expr, sample_info = si, variable_info = vi); validObject(od)'
```

## Evaluation signals

- validObject() returns TRUE for the constructed mass_dataset instance
- After every subsetting/filtering operation, nrow(sample_info) and nrow(variable_info) match dim(expression_data) exactly (samples in columns, features in rows)
- Removing samples from expression_data prunes the corresponding sample_info rows in the same order; removing features prunes the corresponding variable_info rows
- Every applied processing step and its parameters are retrievable from the process_info slot, so the step sequence can be replayed/inspected
- sessionInfo() output is recorded so the validated environment is reproducible

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass, so the mass_dataset container may need conversion to other tools' data structures
- The tidymass package is marked experimental (lifecycle badge) in the project README
- The uniform container must be maintained by the user when extending the pipeline; existing whole-workflow tools were criticized precisely for lacking a uniform data structure, so ad-hoc extensions risk breaking slot synchronization

## Evidence

- [abstract] A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [intro] TidyMass is an object-oriented workflow with a uniform data structure "mass_dataset" that is the kernel of tidyMass: [evidence span withheld — anchor status: not_found]
- [intro] tidyMass has high interoperability with other existing tools, converting "mass_dataset" class with other data structures required by other tools: [evidence span withheld — anchor status: not_found]
- [methods] You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass): "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [readme] remotes::install_gitlab("tidymass/tidymass"): "remotes::install_gitlab("tidymass/tidymass")"
