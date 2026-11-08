---
name: mass-dataset-interoperability-conversion
description: Use when a mass_dataset object produced by tidyMass must be consumed
  by tools outside the tidyMass ecosystem, or when downstream analysis requires SummarizedExperiment,
  mzTab-M, or MetDNA-compatible files.
when_to_use_negative:
- The analysis will stay entirely within the tidyMass ecosystem — conversion is only
  needed to hand data to tools that use different data structures.
- Input is raw LC-MS data (e.g. mzML) that has not yet been processed into a mass_dataset
  object; tidyMass data processing must come first.
- The target tool's required data structure is not covered by the available converters
  (the paper notes some widely used metabolomics tools are not yet wrapped or supported,
  requiring conversion to 'more eligible data structures' in the future).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: tidyMass
  role: Provides the mass_dataset class and the R-based computational framework whose
    uniform data structure is the source of every conversion
  repo: https://www.tidymass.org/
- name: massdataset
  role: R package supplying the conversion functions that export mass_dataset objects
    to SummarizedExperiment, mzTab-M and MetDNA formats
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: SummarizedExperiment
  role: Target data structure for Bioconductor-based downstream analysis such as MetaboAnnotation
- name: MetaboAnnotation
  role: Consumer of the converted SummarizedExperiment object for annotation workflows
- name: mzTab-M
  role: Target interchange format for metabolomics results written to file
- name: MetDNA
  role: External tool whose required input format is produced by the MetDNA-format
    converter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:2484eb670ff34ae83624a32c4579e422dc2935cf2ca2be3acb46d75754d8d275
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA
  - MetaboAnnotation
  - R
  - SummarizedExperiment
  - massdataset
  - mzTab-M
  - tidyMass
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-dataset-interoperability-conversion

## Summary

Converts the tidyMass 'mass_dataset' object — the uniform data structure at the kernel of tidyMass — into data structures used by external metabolomics tools (SummarizedExperiment for MetaboAnnotation, mzTab-M, and MetDNA formats) to achieve high interoperability with existing LC-MS workflows.

## When to use

Use when a mass_dataset object produced by tidyMass must be consumed by tools outside the tidyMass ecosystem, or when downstream analysis requires SummarizedExperiment, mzTab-M, or MetDNA-compatible files.

## When NOT to use

- The analysis will stay entirely within the tidyMass ecosystem — conversion is only needed to hand data to tools that use different data structures.
- Input is raw LC-MS data (e.g. mzML) that has not yet been processed into a mass_dataset object; tidyMass data processing must come first.
- The target tool's required data structure is not covered by the available converters (the paper notes some widely used metabolomics tools are not yet wrapped or supported, requiring conversion to 'more eligible data structures' in the future).

## Inputs

- mass_dataset object (tidyMass uniform data structure containing expression/feature table, sample metadata and feature annotation)
- tidyMass case-study data or massdataset interoperability vignette example data

## Outputs

- SummarizedExperiment RDS object (usable with MetaboAnnotation)
- mzTab-M format file
- MetDNA format file

## How to apply

Load a mass_dataset object (e.g. from the tidyMass case study or the massdataset interoperability vignette) in R. Apply the massdataset package's conversion functions to (1) build a SummarizedExperiment object suitable for MetaboAnnotation, (2) serialize the object to mzTab-M and write it to file, and (3) serialize it to the MetDNA format. The rationale, per the paper, is that tidyMass stores processed metabolomics data and processing parameters in a uniform, traceable format, so conversion out of that format lets other tools operate on the same expression/feature table, sample metadata and feature annotation without re-processing raw LC-MS data. Finally inspect each converted artifact (print the SummarizedExperiment; read back the mzTab-M and MetDNA files) and save the three artifacts as outputs for interoperability verification.

## Related tools

- **tidyMass** (Provides the mass_dataset class and the R-based computational framework whose uniform data structure is the source of every conversion) — https://www.tidymass.org/
- **massdataset** (R package supplying the conversion functions that export mass_dataset objects to SummarizedExperiment, mzTab-M and MetDNA formats) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **SummarizedExperiment** (Target data structure for Bioconductor-based downstream analysis such as MetaboAnnotation)
- **MetaboAnnotation** (Consumer of the converted SummarizedExperiment object for annotation workflows)
- **mzTab-M** (Target interchange format for metabolomics results written to file)
- **MetDNA** (External tool whose required input format is produced by the MetDNA-format converter)

## Examples

```
se <- convert_mass_dataset_to_se(mass_dataset_obj); convert_mass_dataset_to_mztab_m(mass_dataset_obj, file = "result.mzTab"); convert_mass_dataset_to_metdna(mass_dataset_obj, file = "result_metdna")
```

## Evaluation signals

- The SummarizedExperiment object prints with the expected dimensions of the original mass_dataset expression/feature table.
- The mzTab-M file reads back without errors and its feature table matches the source mass_dataset.
- The MetDNA-format file reads back and preserves the sample metadata and feature annotation.
- All three artifacts (SummarizedExperiment RDS, mzTab-M file, MetDNA file) are saved as final outputs for interoperability verification.
- Conversion preserves the expression/feature table, sample metadata and feature annotation across all three formats.

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; the paper states it 'is capable to convert the "mass_dataset" class to more eligible data structures for these tools in the future', so converters for other target structures may be missing.
- Conversion moves data out of the mass_dataset structure that stores processing parameters, so traceability of prior analysis steps may not carry into the exported formats.

## Evidence

- [intro] tidyMass has high interoperability with other existing tools, converting "mass_dataset" class with other data structures required by other tools: [evidence span withheld — anchor status: not_found]
- [abstract] uniform data format stores processed data and parameters: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters, making it possible to trace the prior analysis steps and"
- [discussion] future converters for unsupported tools: "some widely used metabolomics tools are not wrapped up or supported in tidyMass, it is capable to convert the “mass_dataset” class to more eligible data structures for these tools in the future."
- [intro] object-oriented workflow with mass_dataset kernel: "the tidyMass is an object-oriented workflow"
- [intro] interoperability vignette reference: "(https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html)"
