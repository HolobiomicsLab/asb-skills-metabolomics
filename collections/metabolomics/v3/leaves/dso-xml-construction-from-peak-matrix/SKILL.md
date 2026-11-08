---
name: dso-xml-construction-from-peak-matrix
description: Use when an aligned, labeled peak table (e.g. from Galaxy-M's Align Samples
  or XCMS-Basic-Birmingham-Pipe) must be converted into the DSO XML format consumed
  by PLS-Toolbox-based cleansing tools.
when_to_use_negative:
- Input is already a DSO XML object from a prior cleansing step
- Data are still raw instrument files (.dat transients, RAW, NetCDF) that have not
  been peak-picked and aligned
- Only a subset of samples will be analyzed and no class label (biological vs blank)
  information exists
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: Galaxy-M
  role: Hosts the 'Create DSO' Galaxy tool that performs the matrix+labels→DSO XML
    conversion
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
- name: PLS-Toolbox
  role: Defines the DSO data structure the XML must encode; downstream cleansing tools
    (Blank Filtering, PQN, KNN, G-log) are PLS-Toolbox multivariate tools
- name: Galaxy
  role: Workflow platform in which the Create DSO tool is executed and chained to
    downstream steps
  repo: https://github.com/galaxyproject/galaxy
- name: XCMS
  role: Upstream LC-MS peak alignment tool whose aligned feature table is a typical
    input to Create DSO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:71967e3f023cbfa5be608db75b73ec76901291ea2492fb3adff68d8587fb3473
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/Galaxy-M
  tools:
  - Galaxy
  - Galaxy-M
  - PLS-Toolbox
  - XCMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# dso-xml-construction-from-peak-matrix

## Summary

Constructs a DSO XML data object that wraps the aligned peak-intensity X matrix with row/column labels and biological-vs-blank class labels, establishing the structured input required by downstream Galaxy-M cleansing steps (Blank Filtering, Sample Filtering, PQN, KNN imputation, G-log transformation).

## When to use

Use when an aligned, labeled peak table (e.g. from Galaxy-M's Align Samples or XCMS-Basic-Birmingham-Pipe) must be converted into the DSO XML format consumed by PLS-Toolbox-based cleansing tools.

## When NOT to use

- Input is already a DSO XML object from a prior cleansing step
- Data are still raw instrument files (.dat transients, RAW, NetCDF) that have not been peak-picked and aligned
- Only a subset of samples will be analyzed and no class label (biological vs blank) information exists

## Inputs

- Aligned X data matrix file (samples × m/z peak intensities, e.g. from Align Samples or XCMS)
- Row label file (sample names)
- Column label file (m/z peak identifiers)
- Class label CSV designating each sample as biological or blank

## Outputs

- DSO XML data object containing the X matrix, row/column labels, and biological-vs-blank class labels

## How to apply

Assemble the aligned X data matrix (rows = samples, columns = m/z peaks) with its row label and column label files, plus a CSV describing whether each sample is biological or blank. Run the Galaxy-M 'Create DSO' tool to combine these into a single DSO XML object. Include only the samples needed for parameter optimization — in the DIMS case study workflow the inclusion input file selected only the QC samples before G-log transformation, because G-log parameters are optimized on QC samples alone. Validate the output XML by confirming it contains the X matrix, labels, and class labels before passing it to Blank Filtering, Sample Filtering, probabilistic quotient normalization, KNN imputation, and G-log transformation tools.

## Related tools

- **Galaxy-M** (Hosts the 'Create DSO' Galaxy tool that performs the matrix+labels→DSO XML conversion) — https://github.com/Viant-Metabolomics/Galaxy-M
- **PLS-Toolbox** (Defines the DSO data structure the XML must encode; downstream cleansing tools (Blank Filtering, PQN, KNN, G-log) are PLS-Toolbox multivariate tools)
- **Galaxy** (Workflow platform in which the Create DSO tool is executed and chained to downstream steps) — https://github.com/galaxyproject/galaxy
- **XCMS** (Upstream LC-MS peak alignment tool whose aligned feature table is a typical input to Create DSO)

## Examples

```
python create_dso.py --xmatrix aligned_peaks.csv --rowlabels rows.csv --collabels cols.csv --classlabels classes.csv --output dso.xml
```

## Evaluation signals

- Output XML parses and contains a populated X matrix with expected dimensions (n samples × m peaks)
- Row and column labels present in the DSO match the input label files sample-for-sample and peak-for-peak
- Class label array distinguishes biological from blank samples consistently with the input CSV
- When the inclusion input selects only QC samples, the DSO contains exactly those samples (DIM case study uses QC-only inclusion before G-log optimization)
- Downstream Blank Filtering / Sample Filtering tools accept the DSO without format errors

## Limitations

- Requires label files and class label CSV to be structurally consistent with the X matrix — mismatches cause silent misalignment downstream
- GigaDB accession for the pre-installed VM was 'to follow' at publication time; installation from GitHub involves complex Ubuntu/WINE setup (32-bit WINEARCH required for MSFileReader-based tools)
- Galaxy-M was tested against a specific Galaxy commit (c429777c93680dcee449fe410f5360afbe673758); other versions may not be compatible

## Evidence

- [abstract] Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank: "Create DSO: combines the X data matrix file with row and column label information and class labels describing whether each sample is biological or blank"
- [readme] Create DSO Galaxy-M tool: "Included are the tool files (both original code and .xml wrappers) for Metabolomics analysis"
- [results] In the DIMS case study workflow, an inclusion input file set only the QC samples to be included before G-log transformation because the G-log parameters are optimized on the QC samples alone.: [evidence span withheld — anchor status: not_found]
- [abstract] Create DSO: combines the X matrix, row/column labels and class labels into a DSO: [evidence span withheld — anchor status: not_found]
- [readme] obtain GalaxyM from github: "Obtain the latest version of GalaxyM from [github](https://github.com/Viant-Metabolomics/Galaxy-M)"
