---
name: ft-icr-ms-peak-filtering-by-mz-error-and-isotopes
description: Use when input is a post-formula-assignment FT-ICR MS peak abundance
  + molecular formula table (e.g. Report.csv) that must be cleaned by m/z range, 13C
  isotopic peaks, 0.5 ppm formula error, and minimum-sample thresholds before downstream
  analysis.
when_to_use_negative:
- The input data set was already filtered by a previous study (as in the S. fallax
  leachate incubation data set, where no peak was filtered because it was previously
  filtered by the original authors)
- Input is still raw FT-ICR MS spectra — molecular formula assignment must be performed
  prior to using MetaboDirect (e.g. with MFAssignR)
- Data comes from a chromatography-coupled MS method rather than direct injection
  FT-ICR MS, which cannot separate chemical isomers and suffers signal suppression/enhancement
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Command-line pipeline whose data pre-processing step performs the m/z, isotope
    (13C), error, and sample-presence filtering and emits the filtered peak/intensity
    tables
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Python package (part of MetaboDirect, written in Python) used for loading
    and filtering the Report.csv and metadata.csv tables
- name: NumPy
  role: Python package used alongside pandas for the numeric filtering and index computation
    in the pre-processing step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:d35a7d363ed451e73b89a3b066f9d11ff675c7d7dcb3dca1f3da1e77d20e879e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ft-icr-ms-peak-filtering-by-mz-error-and-isotopes

## Summary

MetaboDirect's data pre-processing step for direct injection FT-ICR MS metabolomic data: filters detected peaks by m/z range, 13C isotopic presence, formula-assignment error, and sample-presence threshold, then assigns compound classes, computes thermodynamic indices, and normalizes intensities.

## When to use

Use when input is a post-formula-assignment FT-ICR MS peak abundance + molecular formula table (e.g. Report.csv) that must be cleaned by m/z range, 13C isotopic peaks, 0.5 ppm formula error, and minimum-sample thresholds before downstream analysis.

## When NOT to use

- The input data set was already filtered by a previous study (as in the S. fallax leachate incubation data set, where no peak was filtered because it was previously filtered by the original authors)
- Input is still raw FT-ICR MS spectra — molecular formula assignment must be performed prior to using MetaboDirect (e.g. with MFAssignR)
- Data comes from a chromatography-coupled MS method rather than direct injection FT-ICR MS, which cannot separate chemical isomers and suffers signal suppression/enhancement

## Inputs

- Report.csv: peak abundance table with assigned molecular formulas from prior processing of raw direct-injection FT-ICR MS spectra (e.g. MFAssignR output)
- metadata.csv: sample metadata table
- User-specified m/z filter range, formula-assignment error threshold (0.5 ppm), and minimum sample-presence count
- Normalization method chosen via the test_normalization companion script

## Outputs

- .csv table of filtered peaks with compound-class assignments
- .csv table of thermodynamic indices per peak (NOSC, GFE, AI_mod, DBE)
- .csv tables of normalized and unnormalized peak intensities across samples

## How to apply

Load the Report.csv (peak abundance and assigned molecular formula table) and metadata.csv with pandas/NumPy. Filter detected peaks by their m/z values using a user-specified range, remove masses whose assigned molecular formulas contain isotopic 13C (this removed ~200 masses from the bacterium-phage example data set), reject peaks with formula-assignment error above 0.5 ppm, and keep only peaks present in at least the user-defined minimum number of samples. Assign compound classes to retained formulas and compute per-peak thermodynamic indices (NOSC, GFE, AI_mod, DBE). Normalize peak intensities across samples using the method selected beforehand by the companion test_normalization (SPANS) script, which can be run before the main MetaboDirect pipeline. Export the filtered peak table with compound-class and thermodynamic-index results, plus normalized and unnormalized intensity tables, as .csv files in the pipeline output directory.

## Related tools

- **MetaboDirect** (Command-line pipeline whose data pre-processing step performs the m/z, isotope (13C), error, and sample-presence filtering and emits the filtered peak/intensity tables) — https://github.com/Coayala/MetaboDirect
- **pandas** (Python package (part of MetaboDirect, written in Python) used for loading and filtering the Report.csv and metadata.csv tables)
- **NumPy** (Python package used alongside pandas for the numeric filtering and index computation in the pre-processing step)

## Examples

```
pip install metabodirect; metabodirect -h
```

## Evaluation signals

- Filtered peak table .csv is generated in the pipeline output directory and is readable
- All masses with assigned formulas containing isotopic carbon are removed (~200 masses removed in the bacterium-phage example)
- No retained peak has formula-assignment error above 0.5 ppm, and all retained peaks fall within the user-specified m/z range
- Every retained peak is present in at least the user-defined minimum number of samples
- Both normalized and unnormalized intensity .csv tables exist and share consistent sample keys with metadata.csv

## Limitations

- Direct injection DI-MS cannot separate chemical isomers and suffers signal suppression or enhancement that can confound downstream data analysis
- Filtering is only triggered on data not previously filtered — e.g. no peak was filtered in the S. fallax data set because the original authors had already filtered it
- MetaboDirect expects molecular formula assignment to have been performed beforehand; it does not assign formulas from raw spectra itself
- Install requires Python >=3.5, R >=4, and Cytoscape >=3.8 with specified Python, R, and Cytoscape libraries

## Evidence

- [intro] m/z, isotope, error and sample-presence filter criteria: [evidence span withheld — anchor status: not_found]
- [results] isotopic-carbon masses removed in example data set: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
- [intro] test_normalization companion script preceding the pipeline: [evidence span withheld — anchor status: not_found]
- [intro] input must be post-formula-assignment peak data: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
- [readme] README scope and installability of the pipeline: "A comprehensive command-line based pipeline for the analysis of direct injection FT-ICR mass spectrometry data."
