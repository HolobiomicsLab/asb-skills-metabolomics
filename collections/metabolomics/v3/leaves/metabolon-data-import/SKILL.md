---
name: metabolon-data-import
description: Use when the input is a Metabolon-platform metabolomics export (Excel
  worksheet such as 'OrigScale') that must be loaded into R as a Metaboprep object
  with sample/feature annotation before quality_control can be run.
when_to_use_negative:
- The input is not a Metabolon-formatted worksheet (e.g. Nightingale NMR, Olink, or
  SomaLogic exports) — those require their respective readers even though metaboprep
  ships example files for them.
- The data have already been loaded into a Metaboprep object and only QC or reporting
  steps remain.
- You only need a plain data frame for custom analysis and do not intend to use the
  metaboprep QC pipeline; a simple list return (return_Metaboprep=FALSE) or generic
  import may suffice.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: metaboprep
  role: Provides read_metabolon to parse Metabolon Excel worksheets and the Metaboprep()
    constructor that builds the object consumed by quality_control
  repo: https://github.com/MRCIEU/metaboprep
- name: readxl
  role: Underlying Excel reader used by read_metabolon to access named sheets such
    as 'OrigScale' in the Metabolon workbook
- name: R
  role: Runtime environment in which read_metabolon and the Metaboprep object class
    are used
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:270b8adc375249901c60ff9d2e680102a0752a063ad5aa72b5e399c02e5c48a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - knitr
  - metaboprep
  - readxl
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metabolon-data-import

## Summary

Import Metabolon-platform metabolomics worksheets into R as a Metaboprep object using metaboprep's read_metabolon, which extracts data from preformed Excel worksheets (e.g. the 'OrigScale' sheet) and prepares them for the standard QC pipeline.

## When to use

Use when the input is a Metabolon-platform metabolomics export (Excel worksheet such as 'OrigScale') that must be loaded into R as a Metaboprep object with sample/feature annotation before quality_control can be run.

## When NOT to use

- The input is not a Metabolon-formatted worksheet (e.g. Nightingale NMR, Olink, or SomaLogic exports) — those require their respective readers even though metaboprep ships example files for them.
- The data have already been loaded into a Metaboprep object and only QC or reporting steps remain.
- You only need a plain data frame for custom analysis and do not intend to use the metaboprep QC pipeline; a simple list return (return_Metaboprep=FALSE) or generic import may suffice.

## Inputs

- Metabolon-platform metabolomics Excel workbook (e.g. metabolon_v1.1_example.xlsx) with a named sheet of raw-scale values ('OrigScale')
- Sheet name string (e.g. 'OrigScale') identifying the measurement worksheet

## Outputs

- Metaboprep object containing input data layer, sample metadata (sample_id, run_day, box_id, lot, reason_excluded, excluded), and feature metadata (feature_id, metabolite_id, platform, pathway, kegg, group_hmdb)
- Alternatively (return_Metaboprep=FALSE): a list with $data, $features, and $samples components for manual Metaboprep() construction

## How to apply

Load the metaboprep package and call read_metabolon on the Metabolon Excel file, setting sheet to the worksheet containing raw peak-area values (e.g. 'OrigScale') and return_Metaboprep=TRUE so a Metaboprep object (with input data plus sample and feature metadata) is returned directly; with return_Metaboprep=FALSE a list is returned whose $data, $features, and $samples components must be passed to the Metaboprep() constructor manually. read_metabolon extracts data from preformed Metabolon worksheets, pairing metabolite measurements with sample annotation columns such as sample_id, run_day, box_id, lot, and reason_excluded. The resulting object is the required input for quality_control(source_layer='input', ...) with thresholds like sample_missingness=0.2 and feature_missingness=0.2. For other platforms (Nightingale NMR, Olink, SomaLogic) metaboprep ships packaged example files (nightingale_v1_example.xlsx, olink_v1_example.txt, somalogic_v1_example.adat), but read_metabolon is specific to the Metabolon format.

## Related tools

- **metaboprep** (Provides read_metabolon to parse Metabolon Excel worksheets and the Metaboprep() constructor that builds the object consumed by quality_control) — https://github.com/MRCIEU/metaboprep
- **readxl** (Underlying Excel reader used by read_metabolon to access named sheets such as 'OrigScale' in the Metabolon workbook)
- **R** (Runtime environment in which read_metabolon and the Metaboprep object class are used)

## Examples

```
mydata <- read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package="metaboprep"), sheet="OrigScale", return_Metaboprep=TRUE)
```

## Evaluation signals

- The returned object is a Metaboprep object whose summary() reports expected Samples and Features counts and Data Layers including 'input'.
- Sample annotation columns include sample_id, run_day, box_id, lot, and reason_excluded; feature annotation includes feature_id, metabolite_id, platform, pathway, kegg, and group_hmdb.
- With return_Metaboprep=FALSE, the returned list exposes $data, $features, and $samples components that can be passed to Metaboprep(data=..., features=..., samples=...) without error.
- The object supports an immediate quality_control(source_layer='input', ...) call without further reshaping, indicating the input layer was correctly populated.

## Limitations

- The package is marked experimental (lifecycle badge), so the reader API may change between versions.
- read_metabolon is specific to Metabolon preformed worksheets; other platforms require different import paths.
- Reading with return_Metaboprep=FALSE returns a list, and forgetting to pass $data/$features/$samples into Metaboprep() will block downstream QC.

## Evidence

- [methods] read_metabolon parses Metabolon-format data: "read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx""
- [readme] read_metabolon with sheet and return_Metaboprep arguments: "mydata <- read_metabolon(system.file("extdata", "metabolon_v1.1_example.xlsx", package = "metaboprep"), sheet = "OrigScale", return_Metaboprep = FALSE"
- [intro] Goal of metaboprep is to read in and process untargeted metabolite data: "Read in and processes (un)targeted metabolite data, saving datasets in tab-delimited format for use elsewhere"
- [abstract] The package extracts data from preformed worksheets and enables selection based on quality metrics: "The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
