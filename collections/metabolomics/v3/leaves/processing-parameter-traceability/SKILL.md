---
name: processing-parameter-traceability
description: Use when a metabolomics pipeline must remain reproducible and auditable
  — i.e., each raw-data-processing, cleaning, or QC step's parameters need to be stored
  alongside the processed feature table so prior steps can be traced.
when_to_use_negative:
- You need to interoperate with tools tidyMass does not wrap or support — although
  the 'mass_dataset' class can be converted to structures required by other tools,
  this skill targets tidyMass-native workflows
- The analysis is a single-shot transformation where no step or parameter history
  needs to be preserved in the output object
- The input is raw vendor instrument data not yet processed into a feature table (raw
  data processing precedes this uniform-format stage)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: tidyMass
  role: Object-oriented R framework whose uniform data structure 'mass_dataset' is
    the kernel that stores processed metabolomics data and processing parameters
  repo: https://www.tidymass.org/
- name: massdataset
  role: Provides the 'mass_dataset' class and its interoperability with other tools'
    data structures
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass (R package)
  role: Installer umbrella package for all tidyMass project packages, installed via
    remotes
  repo: https://gitlab.com/tidymass/tidymass
- name: remotes
  role: Used to install tidymass from GitLab or GitHub
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:66d908b3fca43f1873ec9ff4cc1b000aa2d7bc46cea473e334156bd0f5f1204f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - massdataset
  - remotes
  - tidyMass
  - tidymass (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# processing-parameter-traceability

## Summary

Encapsulates LC-MS untargeted metabolomics processing so that every data-cleaning and analysis step is recorded in a uniform data object ('mass_dataset'), enabling prior analysis steps and their parameters to be traced.

## When to use

Use when a metabolomics pipeline must remain reproducible and auditable — i.e., each raw-data-processing, cleaning, or QC step's parameters need to be stored alongside the processed feature table so prior steps can be traced.

## When NOT to use

- You need to interoperate with tools tidyMass does not wrap or support — although the 'mass_dataset' class can be converted to structures required by other tools, this skill targets tidyMass-native workflows
- The analysis is a single-shot transformation where no step or parameter history needs to be preserved in the output object
- The input is raw vendor instrument data not yet processed into a feature table (raw data processing precedes this uniform-format stage)

## Inputs

- LC-MS untargeted metabolomics feature/expression data (peak tables from raw data processing)
- sample metadata
- variable metadata
- ms2_data and annotation_table components
- parameters of each processing step (data cleaning, QC, annotation, statistics)

## Outputs

- 'mass_dataset' object with embedded processing parameters
- traceable record of prior analysis steps and their parameters
- processed feature table aligned with sample_info and variable_info

## How to apply

Create or work with a 'mass_dataset' object — the kernel of tidyMass — which stores the expression data (metabolic feature table), sample metadata, variable metadata, MS2 data, annotation table, and processing parameters. After every processing operation (e.g., removing noisy features, filtering annotations, statistical analysis), write the step and its parameters into the object's processing-parameters record rather than discarding them, so the full chain of prior analysis steps is traceable inside the object itself. Apply operations through the tidyMass packages (installed via remotes from GitLab or GitHub) so the uniform data structure, rather than loose scripts, carries the parameter history forward. Verify traceability by inspecting the recorded parameters of earlier steps in the returned object and confirming the uniform format persisted across the workflow.

## Related tools

- **tidyMass** (Object-oriented R framework whose uniform data structure 'mass_dataset' is the kernel that stores processed metabolomics data and processing parameters) — https://www.tidymass.org/
- **massdataset** (Provides the 'mass_dataset' class and its interoperability with other tools' data structures) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass (R package)** (Installer umbrella package for all tidyMass project packages, installed via remotes) — https://gitlab.com/tidymass/tidymass
- **remotes** (Used to install tidymass from GitLab or GitHub) — https://github.com/tidymass/tidymass

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- The returned object is a valid 'mass_dataset' and contains processing parameters for each step applied
- Prior analysis steps can be traced by reading the stored parameters without consulting external scripts or session logs
- Expression data, sample metadata, and variable metadata remain dimension-consistent after each filtering/subsetting step
- The uniform data format persists across the full workflow (raw data processing → data cleaning → QC → annotation → statistics)
- The Docker case-study image (jaspershen/tidymass-case-study) reproduces the recorded data, code, and analysis environment

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; conversion of the 'mass_dataset' class to other tools' data structures is needed for those cases
- Existing whole-workflow tools' limitation of adopting neither modular design nor a uniform data structure is what motivates this approach — mixing with non-uniform tool outputs can break traceability

## Evidence

- [abstract] A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [intro] TidyMass is an object-oriented workflow with a uniform data structure "mass_dataset" that is the kernel of tidyMass: "the tidyMass is an object-oriented workflow"
- [intro] tidyMass has high interoperability with other existing tools, converting "mass_dataset" class with other data structures required by other tools: "tidyMass has high interoperability with other existing tools"
- [methods] You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass): "You can install `tidymass` from [GitLab](https://gitlab.com/tidymass/tidymass)"
- [readme] `tidymass` is a R package which is used to manage and organize all the pacakges in [tidyMass project](https://www.tidymass.org/): [evidence span withheld — anchor status: external_doc]
