---
name: thermodynamic-gibbs-index-calculation
description: Use when an FT-ICR MS peak table with assigned molecular formulas (e.g.,
  Report.csv) needs per-peak thermodynamic indices such as NOSC/GFE for energetic
  or chemodiversity analysis.
when_to_use_negative:
- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed
  prior to using MetaboDirect
- The peak table was already filtered upstream (e.g., the S. fallax leachate data
  set, where no peak was filtered because it was previously filtered by the original
  authors)
- Analysis requires separation of chemical isomers — direct injection FT-ICR MS cannot
  separate isomers and suffers signal suppression/enhancement
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboDirect
  role: Pipeline whose data pre-processing step filters peaks and computes the thermodynamic
    indices per molecular formula
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads the Report.csv peak/formula table and metadata.csv, and writes the .csv
    outputs of filtered peaks with indices
- name: NumPy
  role: Numeric backend used by MetaboDirect (Python) for the index calculations over
    peak/formula arrays
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:b33863835466c9a066067233dcf7f71893ccb3f659a962601735f1c9b8f355d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# thermodynamic-gibbs-index-calculation

## Summary

Computes per-peak thermodynamic and structural descriptors (NOSC, GFE, AI_mod, DBE) from the molecular formulas of an FT-ICR MS peak table as part of the MetaboDirect data pre-processing step, enabling chemodiversity and energetic-degradation interpretation.

## When to use

Use when an FT-ICR MS peak table with assigned molecular formulas (e.g., Report.csv) needs per-peak thermodynamic indices such as NOSC/GFE for energetic or chemodiversity analysis.

## When NOT to use

- Input is raw FT-ICR MS spectra — molecular formula assignment must be performed prior to using MetaboDirect
- The peak table was already filtered upstream (e.g., the S. fallax leachate data set, where no peak was filtered because it was previously filtered by the original authors)
- Analysis requires separation of chemical isomers — direct injection FT-ICR MS cannot separate isomers and suffers signal suppression/enhancement

## Inputs

- Report.csv — peak abundance table with assigned molecular formulas produced after initial processing of raw FT-ICR MS spectra
- metadata.csv — sample metadata table
- User-specified m/z range, formula-assignment error threshold (0.5 ppm), and minimum number of samples

## Outputs

- Filtered peak table with computed thermodynamic indices (NOSC, GFE, AI_mod, DBE) and compound classes
- .csv files of filtered peaks with normalized and unnormalized intensities in the pipeline output directory

## How to apply

Load the peak abundance and assigned molecular formula table (Report.csv) plus metadata.csv with pandas, after the peaks have been filtered by m/z range, removal of 13C isotopic formulas, formula-assignment error ≤ 0.5 ppm, and presence in the user-defined minimum number of samples. For each retained molecular formula (e.g., CHO, CHOS, CHON elemental classes), compute the nominal oxidation state of carbon (NOSC), Gibbs free energy (GFE), aromaticity index (AI_mod), and double bond equivalent (DBE) descriptors. Also assign compound classes to the retained formulas so indices can be interpreted per class. Retain both normalized (method selected via the test_normalization companion script) and unnormalized intensities alongside the computed indices, and export the result as .csv in the pipeline output directory. Validation: in the bacterium-phage example, ~200 masses were removed because they contained an isotopic carbon, so the index-bearing table should be smaller than the input Report.csv.

## Related tools

- **MetaboDirect** (Pipeline whose data pre-processing step filters peaks and computes the thermodynamic indices per molecular formula) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads the Report.csv peak/formula table and metadata.csv, and writes the .csv outputs of filtered peaks with indices)
- **NumPy** (Numeric backend used by MetaboDirect (Python) for the index calculations over peak/formula arrays)

## Examples

```
import pandas as pd; df = pd.read_csv('Report.csv'); meta = pd.read_csv('metadata.csv')
```

## Evaluation signals

- A filtered peak table is generated and is smaller than the input Report.csv
- Isotopic-carbon-containing masses are removed — in the bacterium-phage example ~200 masses were filtered out for containing an isotopic carbon
- All retained formulas satisfy the ≤ 0.5 ppm formula-assignment error threshold and the user-defined minimum sample-presence threshold
- Every retained molecular formula carries finite NOSC, GFE, AI_mod, and DBE values plus a compound-class assignment
- Exported .csv files exist, are readable with pandas, and contain both normalized and unnormalized intensity columns

## Limitations

- Direct injection FT-ICR MS cannot separate chemical isomers, and signal suppression or enhancement can confound downstream data analysis
- Index computation requires prior molecular formula assignment; peaks without assigned formulas cannot receive thermodynamic descriptors
- The best intensity normalization method is data-dependent and should be selected first with the test_normalization companion script

## Evidence

- [intro] Filtering criteria: [evidence span withheld — anchor status: not_found]
- [intro] Pipeline structure: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration"
- [intro] Input requirements: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra"
- [results] Isotopic filtering outcome: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
- [results] Normalization selection: "the first step was to use the “test_normalization” companion script to help determine which normalization method worked the best"
- [readme] Software stack: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
