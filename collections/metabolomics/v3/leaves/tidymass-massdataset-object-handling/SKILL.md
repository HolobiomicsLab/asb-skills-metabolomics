---
name: tidymass-massdataset-object-handling
description: Use when LC-MS untargeted metabolomics results (feature tables, differential-metabolite
  lists, annotations) must be carried between tidyMass modules or converted to/from
  external tool formats, e.g. feeding sex-stratified differential metabolites into
  metpath enrichment.
when_to_use_negative:
- The input is still raw instrument data (e.g. mzML/NetCDF) needing peak picking and
  grouping — use a preprocessing engine such as xcms or the tidyMass raw-data-processing
  module instead of object handling
- The downstream tool has no mass_dataset interoperability converter — the paper notes
  some widely used metabolomics tools are not wrapped up or supported in tidyMass
- The analysis is not LC-MS-based untargeted metabolomics data processing/analysis,
  tidyMass's declared scope
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: massdataset
  role: Provides the mass_dataset class and converters to interoperate with other
    tools' data structures
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Metapackage that installs and organizes all tidyMass project packages (massdataset,
    massstat, metpath) sharing one data structure
  repo: https://gitlab.com/tidymass/tidymass
- name: metpath
  role: Downstream consumer of mass_dataset-derived differential metabolite lists
    for KEGG/SMPDB pathway enrichment
- name: xcms
  role: External preprocessing alternative whose outputs can be converted into mass_dataset
    structures via massdataset interoperability
  repo: https://github.com/sneumann/xcms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:5fef5ab98a4edffe5bc5a7e3f0cdffb206bc1441e603764677be3bec9843d42f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dplyr
  - ggplot2
  - massdataset
  - massstat
  - metpath
  - remotes
  - tidyMass
  - tidymass
  - xcms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# tidymass-massdataset-object-handling

## Summary

Create, convert, and pass tidyMass 'mass_dataset' objects (the uniform data structure that is the kernel of tidyMass) so that downstream packages such as massstat and metpath can consume the same processed metabolomics data and processing parameters within one reproducible R workflow.

## When to use

Use when LC-MS untargeted metabolomics results (feature tables, differential-metabolite lists, annotations) must be carried between tidyMass modules or converted to/from external tool formats, e.g. feeding sex-stratified differential metabolites into metpath enrichment.

## When NOT to use

- The input is still raw instrument data (e.g. mzML/NetCDF) needing peak picking and grouping — use a preprocessing engine such as xcms or the tidyMass raw-data-processing module instead of object handling
- The downstream tool has no mass_dataset interoperability converter — the paper notes some widely used metabolomics tools are not wrapped up or supported in tidyMass
- The analysis is not LC-MS-based untargeted metabolomics data processing/analysis, tidyMass's declared scope

## Inputs

- Processed LC-MS untargeted metabolomics feature tables (intensity matrix with sample and feature metadata)
- Differential-metabolite tables from a massstat differential-analysis step (e.g. male-tumor vs female-tumor lists)
- Metabolite annotations restricted to MSI level 2
- Sample metadata / group labels

## Outputs

- mass_dataset objects carrying expression data, sample info, variable info, and processing parameters
- Converted data structures for external tools (e.g. data frames of differential metabolites for metpath)
- Traceable records of prior analysis steps and parameters stored alongside the data

## How to apply

Install the tidyMass ecosystem with remotes from GitLab or GitHub (remotes::install_gitlab("tidymass/tidymass")), which pulls the modular packages including massdataset, massstat and metpath. Represent every processed data set as a 'mass_dataset' object — the uniform format designed to store and manage processed metabolomics data plus processing parameters, making prior analysis steps traceable. Before handing data downstream, apply the pipeline's QC filters: drop features with more than 20% missing values in QC samples or more than 50% MVs in study groups, and keep only level-2 annotations (removing redundant annotations by annotation score). When an external tool requires a different structure, use massdataset's interoperability converters to convert the 'mass_dataset' class to the format that tool needs (e.g. exporting differential-metabolite tables for metPath's enrich_kegg()/enrich_smpdb() over-representation testing). The rationale is tidyMass's modular, object-oriented design: shared grammar and data structure let users integrate their own pipeline steps without reformatting.

## Related tools

- **massdataset** (Provides the mass_dataset class and converters to interoperate with other tools' data structures) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Metapackage that installs and organizes all tidyMass project packages (massdataset, massstat, metpath) sharing one data structure) — https://gitlab.com/tidymass/tidymass
- **metpath** (Downstream consumer of mass_dataset-derived differential metabolite lists for KEGG/SMPDB pathway enrichment)
- **xcms** (External preprocessing alternative whose outputs can be converted into mass_dataset structures via massdataset interoperability) — https://github.com/sneumann/xcms

## Examples

```
remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- The object carries class 'mass_dataset' and contains expression data plus sample_info, variable_info, and processing-parameter slots
- Traceability check: prior analysis steps and parameters stored in the object can be retraced (the stated purpose of the uniform format)
- Round-trip conversion (mass_dataset -> external structure -> mass_dataset) preserves feature intensities and sample metadata
- Upstream QC invariants hold: no feature with >20% MVs in QC samples or >50% MVs in study groups; only level-2 annotations retained
- Downstream calls (e.g. metpath enrich_kegg/enrich_smpdb) accept the exported metabolite list without reformatting errors

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; the mass_dataset class must be converted to their eligible data structures
- tidymass package itself is marked experimental (lifecycle badge) in the README
- Only level-2 annotations were used in the reference workflow, so level-3/4 annotations carried in the object should not feed downstream analysis

## Evidence

- [intro] TidyMass is an object-oriented workflow with a uniform data structure "mass_dataset" that is the kernel of tidyMass: "the tidyMass is an object-oriented workflow"
- [intro] tidyMass has high interoperability with other existing tools, converting "mass_dataset" class with other data structures required by other tools: [evidence span withheld — anchor status: not_found]
- [abstract] A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [readme] tidymass is an R package which is used to manage and organize all the packages in the tidyMass project, installable via remotes::install_gitlab("tidymass/tidymass"): [evidence span withheld — anchor status: external_doc]
