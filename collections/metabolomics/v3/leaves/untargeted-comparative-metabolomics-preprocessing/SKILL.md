---
name: untargeted-comparative-metabolomics-preprocessing
description: Use when raw centroided LC-HRMS/MS files (mzML/mzXML) from a comparative
  metabolomics study (e.g. mutant vs. wildtype) need feature detection, alignment,
  blank removal, and peak-quality filtering into an analysis-ready feature table.
when_to_use_negative:
- The data are already an exported, quality-filtered feature table (e.g. .mskFT or
  .csv with mz/rt columns) rather than raw centroided MS files — preprocessing would
  duplicate completed work
- The analysis goal is downstream structural characterization or molecular formula
  prediction (e.g. SIRIUS, CAMERA annotation, Seven Golden Rules filtering) rather
  than feature detection and filtering
- MS data are profile mode rather than centroided, or come from a non-LC/MS modality
  not supported by the xcms centWave workflow
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metaboseek
  role: Open-source shiny-based GUI platform that hosts the XCMS module, blank subtraction,
    and Fast Peak Shapes / Peak Quality filtering and exports the filtered feature
    table
  repo: https://github.com/mjhelf/Metaboseek
- name: xcms
  role: R package performing centWave feature detection and feature alignment across
    samples via Metaboseek's graphical user interface
  repo: https://github.com/sneumann/xcms
- name: MSnbase
  role: Bioconductor package used with xcms to load xcms results into the Metaboseek
    app as a Feature Table
- name: mzR
  role: Underlying parser for reading centroided mzML/mzXML raw MS files into the
    R environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28391-9
    title: Comparative metabolomics with Metaboseek reveals functions of a conserved
      fat metabolism pathway in C. elegans
  source_content_hash: sha256:f954afa7a793174de5c29d12a5064b8ad1c8d3d5fae5e81ced439e1968d587ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjhelf/Metaboseek
  tools:
  - MSnbase
  - Metaboseek
  - mzR
  - xcms
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# untargeted-comparative-metabolomics-preprocessing

## Summary

Preprocess untargeted HPLC-HRMS/MS comparative metabolomics data by detecting and aligning molecular features with the XCMS module of Metaboseek, then reduce raw feature lists (e.g. >700,000 features) via blank subtraction (10-fold over blank) and Peak Quality thresholding to a reliable filtered table (~150,000 features).

## When to use

Use when raw centroided LC-HRMS/MS files (mzML/mzXML) from a comparative metabolomics study (e.g. mutant vs. wildtype) need feature detection, alignment, blank removal, and peak-quality filtering into an analysis-ready feature table.

## When NOT to use

- The data are already an exported, quality-filtered feature table (e.g. .mskFT or .csv with mz/rt columns) rather than raw centroided MS files — preprocessing would duplicate completed work
- The analysis goal is downstream structural characterization or molecular formula prediction (e.g. SIRIUS, CAMERA annotation, Seven Golden Rules filtering) rather than feature detection and filtering
- MS data are profile mode rather than centroided, or come from a non-LC/MS modality not supported by the xcms centWave workflow

## Inputs

- Centroided LC-HRMS/MS data files (.mzML or .mzXML) from sample and blank injections
- Blank injection files for blank subtraction
- Feature table (.mskFT) preserving the complete xcms processing history

## Outputs

- Filtered feature table (feature_table.csv, comma-separated) with required `mz` column and `rt` column in seconds (~150,000 rows from >700,000 detected features)
- Metaboseek project folder storing run settings and output feature tables
- .mskFT feature table file with complete processing history

## How to apply

Load centroided MS data (.mzML or .mzXML) into Metaboseek individually or from a project folder, then run the XCMS module with centWave feature detection under Metaboseek_default settings to detect and align molecular features (defined by m/z and retention time) across all samples; the run creates a project folder storing settings and output feature tables. Reload the resulting feature table as an .mskFT file to preserve the complete processing history of the xcms run. Apply blank subtraction by removing features whose intensity is not at least 10-fold higher than in blank injections, then apply Fast Peak Shapes / Peak Quality thresholding to discard poorly shaped or low-quality peaks. Export the filtered feature table with the Save Table function in comma-separated .csv format containing a column `mz` and a column `rt` in seconds so it can be reloaded into Metaboseek. In the reference study this pipeline reduced more than 700,000 combined features from hacl-1 mutant vs. WT C. elegans exo- and endo-metabolomes to approximately 150,000 features.

## Related tools

- **Metaboseek** (Open-source shiny-based GUI platform that hosts the XCMS module, blank subtraction, and Fast Peak Shapes / Peak Quality filtering and exports the filtered feature table) — https://github.com/mjhelf/Metaboseek
- **xcms** (R package performing centWave feature detection and feature alignment across samples via Metaboseek's graphical user interface) — https://github.com/sneumann/xcms
- **MSnbase** (Bioconductor package used with xcms to load xcms results into the Metaboseek app as a Feature Table)
- **mzR** (Underlying parser for reading centroided mzML/mzXML raw MS files into the R environment)

## Examples

```
devtools::install_github("mjhelf/Metaboseek")
```

## Evaluation signals

- feature_table.csv exists, parses as comma-separated values, and contains the required `mz` column and `rt` column in seconds so it can be reloaded into Metaboseek
- Row count of the filtered table is consistent with the expected filtered feature set (~150,000 features expected from the full >700,000-feature study dataset)
- No retained feature has an intensity less than 10-fold higher than in blank injections (blank-subtraction invariant)
- The .mskFT file reloads into Metaboseek and reproduces the complete processing history of the xcms run
- All retained peaks pass the Fast Peak Shapes / Peak Quality thresholds (no poorly shaped or low-quality peaks remain)

## Limitations

- Metaboseek is under active development with new features constantly added; settings and behavior may change between versions (README notes the download-page change log)
- The workflow requires centroided data; it is built around the xcms centWave feature-detection model and Metaboseek_default settings, so substantially different acquisition styles may need parameter adjustment
- Untargeted feature tables remain largely unannotated — the paper notes metabolite annotations are limited to confidence Level 3 (tentative structure or compound class assignments) and many detected features remain unidentified

## Evidence

- [results] Untargeted HPLC-HRMS/MS analysis yielded >700,000 features reduced to ~150,000: [evidence span withheld — anchor status: not_found]
- [methods] Metaboseek GUI for xcms feature detection and alignment: "graphical user interface to set up data analysis with the `xcms` package to detect and align molecular features from LC/MS data"
- [methods] Load xcms results as a Feature Table using xcms and MSnbase: "You can then load xcms results into the app as a "Feature Table" (using `xcms` and [`MSnbase`](https://bioconductor.org/packages/release/bioc/html/MSnbase.html) packages"
- [readme] Metaboseek is built on xcms functions with a shiny GUI: "It is built on functions from the <a href = "https://github.com/sneumann/xcms">xcms</a> package, and provides a web-based graphical user interface built with <a href ="
- [intro] Three major steps of metabolomics data processing: "Processing of metabolomics data involves three major steps – feature detection, comparative statistical analysis, and structural characterization"
