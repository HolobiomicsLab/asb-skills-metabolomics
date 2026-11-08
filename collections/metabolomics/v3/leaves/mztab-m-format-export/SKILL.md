---
name: mztab-m-format-export
description: Use when a processed metabolomics feature table held as a mass_dataset
  object must be shared with, or ingested by, external tools that require the mzTab-M
  data structure.
when_to_use_negative:
- The data has not been assembled into a mass_dataset object yet (raw vendor files
  or unprocessed peak lists should first go through tidyMass data processing)
- The target tool is already part of the tidyMass ecosystem and consumes mass_dataset
  objects natively, making a format round-trip unnecessary
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: tidyMass
  role: Object-oriented LC-MS workflow whose uniform mass_dataset class is the source
    object for mzTab-M export
  repo: https://www.tidymass.org/
- name: massdataset
  role: Provides the conversion functions that transform a mass_dataset object into
    the mzTab-M format and other data structures
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: R
  role: Language in which the conversion functions are executed and the mzTab-M file
    is written and read back
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:73699976db4febfcd5b7f8bf9a2e08ed4644e2233a01bfe37b8c34c3dd5ac7fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnnotation
  - R
  - massdataset
  - mzTab-M
  - tidyMass
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mzTab-M format export

## Summary

Exports a tidyMass 'mass_dataset' object to the mzTab-M reporting format (alongside SummarizedExperiment and MetDNA conversions) so processed LC-MS metabolomics data can be exchanged with tools outside the tidyMass ecosystem.

## When to use

Use when a processed metabolomics feature table held as a mass_dataset object must be shared with, or ingested by, external tools that require the mzTab-M data structure.

## When NOT to use

- The data has not been assembled into a mass_dataset object yet (raw vendor files or unprocessed peak lists should first go through tidyMass data processing)
- The target tool is already part of the tidyMass ecosystem and consumes mass_dataset objects natively, making a format round-trip unnecessary

## Inputs

- mass_dataset object (tidyMass uniform data structure holding expression/feature table, sample metadata, and feature annotation)

## Outputs

- mzTab-M format file (written to disk)
- Optionally, companion artifacts for cross-checking: SummarizedExperiment RDS and MetDNA format file produced from the same mass_dataset object

## How to apply

Start from a mass_dataset object (e.g. from the tidyMass case-study Docker image or the massdataset interoperability vignette example data) so that the expression/feature table, sample metadata, and feature annotation all travel with the object. Use the massdataset conversion functions to convert the mass_dataset object to the mzTab-M format and write it to file. Inspect the written mzTab-M file by reading it back to confirm the conversion preserves the feature table, sample metadata, and feature annotation. This export is what underpins tidyMass's stated interoperability strategy: the mass_dataset class can be converted to mzTab-M, MetDNA, and SummarizedExperiment (for MetaboAnnotation) formats, so the same source object can be verified across all three conversions.

## Related tools

- **tidyMass** (Object-oriented LC-MS workflow whose uniform mass_dataset class is the source object for mzTab-M export) — https://www.tidymass.org/
- **massdataset** (Provides the conversion functions that transform a mass_dataset object into the mzTab-M format and other data structures) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **R** (Language in which the conversion functions are executed and the mzTab-M file is written and read back)

## Examples

```
if(!require(remotes)){install.packages("remotes")}; remotes::install_gitlab("tidymass/tidymass")
```

## Evaluation signals

- The written mzTab-M file can be read back without errors
- The feature/expression table in the mzTab-M file matches the mass_dataset expression table (feature and sample identifiers preserved)
- Sample metadata is present and consistent with the source mass_dataset object
- Feature annotation is carried through to the mzTab-M representation
- The same mass_dataset object also converts cleanly to SummarizedExperiment and MetDNA formats, confirming the source object was complete before export

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; conversion coverage to 'more eligible data structures for these tools' is planned for the future
- Interoperability depends on the mass_dataset object retaining processing parameters and prior analysis steps, since the uniform format is what makes prior analysis traceable

## Evidence

- [intro] tidyMass has high interoperability with other existing tools: [evidence span withheld — anchor status: not_found]
- [discussion] future conversion coverage: "some widely used metabolomics tools are not wrapped up or supported in tidyMass, it is capable to convert the “mass_dataset” class to more eligible data structures for these tools in the future."
- [intro] interoperability vignette: "(https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html)"
- [readme] tidymass package role: [evidence span withheld — anchor status: external_doc]
