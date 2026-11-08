---
name: summarizedexperiment-object-construction
description: Use when a mass_dataset object (from tidyMass processing or the massdataset
  interoperability vignette) must be handed off to tools that require a SummarizedExperiment
  data structure.
when_to_use_negative:
- The input is already a SummarizedExperiment or another target format — conversion
  is redundant.
- The goal is raw LC-MS data processing (peak picking/grouping), which is handled
  by xcms, MZmine or MS-DIAL before mass_dataset construction.
- Downstream tooling accepts mass_dataset natively, i.e. within the tidyMass ecosystem
  itself.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: tidyMass
  role: Object-oriented LC-MS workflow whose uniform mass_dataset class is the source
    object for conversion
  repo: https://github.com/tidymass/tidymass
- name: massdataset
  role: Package providing the conversion functions from mass_dataset to SummarizedExperiment,
    mzTab-M and MetDNA formats
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: SummarizedExperiment
  role: Target Bioconductor data structure produced by the conversion
- name: MetaboAnnotation
  role: Downstream annotation tool that consumes the converted SummarizedExperiment
    object
- name: mzTab-M
  role: Interoperable export format for the converted feature table and metadata
- name: MetDNA
  role: Alternative export format supported by mass_dataset conversion
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:66d45881fd9a252cc82a4108fa1495a28b3f6f12607216e5bf3ed76fa37cb6e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tidymass/tidymass
  tools:
  - MetDNA
  - MetaboAnnotation
  - R
  - SummarizedExperiment
  - faahKO
  - massdataset
  - mzTab-M
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# SummarizedExperiment object construction

## Summary

Constructs a Bioconductor SummarizedExperiment object from a tidyMass mass_dataset object so that the processed LC-MS feature table, sample metadata and feature annotation can be consumed by SummarizedExperiment-based tools such as MetaboAnnotation, alongside mzTab-M and MetDNA exports.

## When to use

Use when a mass_dataset object (from tidyMass processing or the massdataset interoperability vignette) must be handed off to tools that require a SummarizedExperiment data structure.

## When NOT to use

- The input is already a SummarizedExperiment or another target format — conversion is redundant.
- The goal is raw LC-MS data processing (peak picking/grouping), which is handled by xcms, MZmine or MS-DIAL before mass_dataset construction.
- Downstream tooling accepts mass_dataset natively, i.e. within the tidyMass ecosystem itself.

## Inputs

- mass_dataset object (expression/feature table, sample metadata, feature annotation)
- massdataset interoperability vignette example data (https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html)
- tidyMass case-study data (docker image jaspershen/tidymass-case-study)

## Outputs

- SummarizedExperiment object (saved as RDS)
- mzTab-M format file
- MetDNA format file

## How to apply

Load the mass_dataset object (e.g. from the tidyMass case study or the massdataset interoperability vignette example data) in R. Call the massdataset conversion function to produce a SummarizedExperiment object in which the assay/feature table, colData (sample metadata) and rowData (feature annotation) are populated from the mass_dataset slots. Then convert the same object to the mzTab-M format and to the MetDNA format and write each to file, since tidyMass is deliberately interoperable with other tools. Inspect each artifact by printing the SummarizedExperiment and reading the mzTab-M and MetDNA files back to confirm the three components were preserved. Save the SummarizedExperiment RDS, the mzTab-M file and the MetDNA file as the interoperability verification outputs.

## Related tools

- **tidyMass** (Object-oriented LC-MS workflow whose uniform mass_dataset class is the source object for conversion) — https://github.com/tidymass/tidymass
- **massdataset** (Package providing the conversion functions from mass_dataset to SummarizedExperiment, mzTab-M and MetDNA formats) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **SummarizedExperiment** (Target Bioconductor data structure produced by the conversion)
- **MetaboAnnotation** (Downstream annotation tool that consumes the converted SummarizedExperiment object)
- **mzTab-M** (Interoperable export format for the converted feature table and metadata)
- **MetDNA** (Alternative export format supported by mass_dataset conversion)

## Examples

```
remotes::install_gitlab("tidymass/tidymass"); # then convert mass_dataset per https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
```

## Evaluation signals

- Printing the resulting SummarizedExperiment object shows assay dimensions equal to the mass_dataset expression table (features x samples).
- colData of the SummarizedExperiment matches the mass_dataset sample metadata; rowData matches the feature annotation.
- The written mzTab-M file can be read back and its feature/sample sections reproduce the source tables.
- The MetDNA file read back preserves the same feature and sample information as the original mass_dataset.
- All three artifacts (SummarizedExperiment RDS, mzTab-M file, MetDNA file) exist and were generated from the same mass_dataset object.

## Limitations

- Some widely used metabolomics tools are not yet wrapped or supported in tidyMass; conversion to structures eligible for those tools is future work.
- The conversion covers mass_dataset-derived processed data, not raw instrument files.

## Evidence

- [intro] tidyMass has high interoperability with other existing tools, converting "mass_dataset" class with other data structures required by other tools: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion: future conversions of mass_dataset to other tool structures: "some widely used metabolomics tools are not wrapped up or supported in tidyMass, it is capable to convert the “mass_dataset” class to more eligible data structures for these tools"
- [readme] README: install tidymass from GitLab: "remotes::install_gitlab("tidymass/tidymass")"
- [abstract] Intro: uniform data format stores processed data and parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
