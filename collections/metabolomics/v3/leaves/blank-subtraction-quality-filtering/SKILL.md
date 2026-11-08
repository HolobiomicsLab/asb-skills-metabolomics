---
name: blank-subtraction-quality-filtering
description: Use when an XCMS feature table from LC-HRMS/MS samples includes blank
  injections and contains low-quality or carryover-derived features that must be removed
  before comparative statistical analysis of sample groups.
when_to_use_negative:
- Input is already a blank-subtracted, quality-filtered feature table — re-filtering
  with a 10-fold blank threshold could discard genuine low-abundance metabolites
- The goal is peak picking on raw vendor files — feature detection with the XCMS centWave
  module must run first; this skill operates on the resulting feature table
- Statistical comparison of groups is already the goal — this skill is a preprocessing
  step, not the downstream comparative statistics module
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Metaboseek
  role: Provides the GUI workflow (Load dialog, blank subtraction, Fast Peak Shapes/Peak
    Quality thresholding, Save Table) used to filter the feature table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: Underlying package, driven through Metaboseek's GUI, that detects and aligns
    the molecular features which are then blank- and quality-filtered
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Package used with xcms to load xcms results into Metaboseek as a Feature Table
    prior to filtering
  repo: https://bioconductor.org/packages/release/bioc/html/MSnbase.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:8bea89a41bf7da37ae21b2ce9994a773d48bfd9a916bea67d941005a898be609
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - mzR
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# blank-subtraction-quality-filtering

## Summary

Reduces an XCMS-detected feature table from untargeted HPLC-HRMS/MS data by removing features attributable to carryover/contaminants via blank subtraction and discarding poorly shaped peaks via Peak Quality thresholding, as used in Metaboseek to reduce >700,000 features from C. elegans hacl-1 vs WT metabolomes to ~150,000 retained features.

## When to use

Use when an XCMS feature table from LC-HRMS/MS samples includes blank injections and contains low-quality or carryover-derived features that must be removed before comparative statistical analysis of sample groups.

## When NOT to use

- Input is already a blank-subtracted, quality-filtered feature table — re-filtering with a 10-fold blank threshold could discard genuine low-abundance metabolites
- The goal is peak picking on raw vendor files — feature detection with the XCMS centWave module must run first; this skill operates on the resulting feature table
- Statistical comparison of groups is already the goal — this skill is a preprocessing step, not the downstream comparative statistics module

## Inputs

- Centroided LC-HRMS/MS data files (.mzML or .mzXML) including sample and blank injections
- XCMS feature table (.mskFT) from centWave feature detection with aligned m/z and retention time features

## Outputs

- Filtered feature table (.csv) with a `mz` column and an `rt` column in seconds (~150,000 features from >700,000 initially detected)
- Feature table session file (.mskFT) preserving the complete processing history of the filtering steps

## How to apply

In Metaboseek, first load the XCMS feature table (.mskFT) produced by centWave feature detection and alignment across all samples, which preserves the processing history of the run. Apply blank subtraction by removing any feature whose intensity is not at least 10-fold higher than in the blank injections, which eliminates features from solvents, column bleed, and carryover. Then apply Fast Peak Shapes / Peak Quality thresholding to discard peaks with irregular or low-quality elution profiles that typically indicate noise or mis-integration. Export the filtered table with Save Table as a .csv containing a `mz` column and an `rt` column in seconds. In the hacl-1 vs WT C. elegans study, these two filters reduced more than 700,000 combined molecular features to approximately 150,000.

## Related tools

- **Metaboseek** (Provides the GUI workflow (Load dialog, blank subtraction, Fast Peak Shapes/Peak Quality thresholding, Save Table) used to filter the feature table) — https://github.com/mjhelf/Metaboseek
- **xcms** (Underlying package, driven through Metaboseek's GUI, that detects and aligns the molecular features which are then blank- and quality-filtered) — https://github.com/sneumann/xcms
- **MSnbase** (Package used with xcms to load xcms results into Metaboseek as a Feature Table prior to filtering) — https://bioconductor.org/packages/release/bioc/html/MSnbase.html

## Examples

```
Rscript -e "devtools::install_github('mjhelf/Metaboseek'); Metaboseek::runMetaboseek()"  # then load the .mskFT feature table, apply blank subtraction (10-fold) and Peak Quality filters, and Save Table as feature_table.csv
```

## Evaluation signals

- feature_table.csv exists, parses as comma-separated values, and contains the required `mz` and `rt` (in seconds) columns so it can be reloaded into Metaboseek
- Row count of the filtered table is consistent with the expected filtered feature set (~150,000 features from the full study dataset of >700,000)
- No retained feature has an intensity less than 10-fold its intensity in blank injections
- Peaks flagged by Fast Peak Shapes / Peak Quality thresholding as poorly shaped are absent from the exported table
- The .mskFT session file preserves the complete processing history of the xcms run and applied filters

## Limitations

- The 10-fold blank threshold and Peak Quality cutoffs are heuristic; overly strict settings may discard genuine low-abundance endogenous metabolites
- Metaboseek is under active development and new features are constantly added, so filter module behavior may change between versions
- Blank subtraction only removes features detected in blanks; contaminants absent from blank injections but present in samples will not be filtered

## Evidence

- [methods] blank subtraction threshold: [evidence span withheld — anchor status: not_found]
- [results] filtering outcome: [evidence span withheld — anchor status: not_found]
- [methods] feature table loading: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] xcms basis in Metaboseek: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
