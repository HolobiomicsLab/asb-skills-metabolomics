---
name: lcms-feature-table-interpretation
description: Use when you have an LC-MS feature table (from XCMS mzXML extraction
  or a custom csv from MS-DIAL/MZmine2) and need to identify which features are in-source
  fragments of which precursor ions.
when_to_use_negative:
- When you already have an ISF-annotated results object and only need the relationship-tree
  visualization (Part 5.2) or the exported ISF result feature table (Part 5.1) rather
  than feature-table interpretation itself
- When your input is raw chromatographic data and you need peak alignment/filling
  across samples — use XCMS directly; ISFrag only operates on the resulting feature
  table
- When you need CAMERA adduct and isotope annotation but your feature table comes
  from a non-XCMS source, since CAMERA annotation is limited to XCMS-only analysis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ISFrag
  role: R package whose find.level3/find.level2/find.level1, ms2.assignment, feature.annotation,
    and get.ISFrag.results functions perform the ISF interpretation of the feature
    table
  repo: https://github.com/HuanLab/ISFrag
- name: XCMS
  role: Used via XCMS.featuretable() to extract the MS1 feature table from mzXML files,
    including peak alignment and filling for multi-sample analyses
- name: MSnbase
  role: Produces the MSnbase object output alongside the dataframe feature table during
    XCMS feature extraction
- name: CAMERA
  role: Provides adduct and isotope annotation, usable only for XCMS-only ISFrag analysis
- name: RStudio
  role: Recommended environment for installing and running the ISFrag package
- name: devtools
  role: Used to install ISFrag from GitHub via install_github('HuanLab/ISFrag')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:b8d6d29256d037bdc7b20c9d05236b9fa7a1ccccdaee628ca9159db12378b454
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag
  tools:
  - CAMERA
  - DiagrammeR
  - ISFrag
  - MSnbase
  - R
  - RStudio
  - XCMS
  - devtools
  - rsvg
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# lcms-feature-table-interpretation

## Summary

Interpret an LC-MS/MS metabolite feature table by recognizing in-source fragments (ISF) with the ISFrag R package, running level 3→2→1 relationship detection and summarizing precursor–fragment relationships. This converts a raw peak table (m/z, rt, rtmin, rtmax, intensities) into annotated ISF relationships usable for results export and tree visualization.

## When to use

Use when you have an LC-MS feature table (from XCMS mzXML extraction or a custom csv from MS-DIAL/MZmine2) and need to identify which features are in-source fragments of which precursor ions.

## When NOT to use

- When you already have an ISF-annotated results object and only need the relationship-tree visualization (Part 5.2) or the exported ISF result feature table (Part 5.1) rather than feature-table interpretation itself
- When your input is raw chromatographic data and you need peak alignment/filling across samples — use XCMS directly; ISFrag only operates on the resulting feature table
- When you need CAMERA adduct and isotope annotation but your feature table comes from a non-XCMS source, since CAMERA annotation is limited to XCMS-only analysis

## Inputs

- mzXML file(s) from DDA, DIA, or fullscan analyses placed in a dedicated folder (for XCMS feature extraction)
- Custom csv feature table with columns strictly ordered: m/z, retention time, min retention time, max retention time, per-sample intensities; retention times in seconds
- Standard spectral library in msp format (e.g. MoNA-export-LC-MS-MS_Negative_Mode.msp) for feature annotation

## Outputs

- Dataframe-formatted MS1 feature table (e.g. xcmsFT with mz, rt, rtmin, rtmax, maxo columns) plus an MSnbase object
- Lists of level 3, level 2, and level 1 in-source fragment relationships
- ISF results object from get.ISFrag.results() summarizing precursor–fragment relationships for export and tree plotting

## How to apply

Generate or import the feature table first: either XCMS.featuretable() on mzXML files from DDA/DIA/fullscan analyses, or custom.featuretable() on a csv whose columns must be exactly m/z, retention time, min rt, max rt, then per-sample intensities with all retention times in seconds. Then run the ISF identification functions sequentially as find.level3(), find.level2(), find.level1() — ISFrag identifies fragments from Level 3 down to Level 1, with the article reporting 100% correct recognition of level 1 ISF features and over 80% for level 2 when tested on metabolite standards. Optionally assign MS2 spectra with ms2.assignment() and annotate against a standard library in msp format (e.g. MoNA-export-LC-MS-MS_Negative_Mode.msp) via feature.annotation(), noting CAMERA adduct/isotope annotation works only for XCMS-only analysis. Finally summarize with get.ISFrag.results() to obtain the results object used for downstream exports. The decision point is the entry route: XCMS extraction versus custom table versus combining both, and whether adduct/isotope annotation is needed.

## Related tools

- **ISFrag** (R package whose find.level3/find.level2/find.level1, ms2.assignment, feature.annotation, and get.ISFrag.results functions perform the ISF interpretation of the feature table) — https://github.com/HuanLab/ISFrag
- **XCMS** (Used via XCMS.featuretable() to extract the MS1 feature table from mzXML files, including peak alignment and filling for multi-sample analyses)
- **MSnbase** (Produces the MSnbase object output alongside the dataframe feature table during XCMS feature extraction)
- **CAMERA** (Provides adduct and isotope annotation, usable only for XCMS-only ISFrag analysis)
- **RStudio** (Recommended environment for installing and running the ISFrag package)
- **devtools** (Used to install ISFrag from GitHub via install_github('HuanLab/ISFrag'))

## Examples

```
xcmsFT <- XCMS.featuretable(MS1directory = MS1directory, type = "single", peakwidth = c(5,20)); ISFresult <- get.ISFrag.results(find.level1(find.level2(find.level3(xcmsFT))))
```

## Evaluation signals

- The feature table dataframe contains the expected columns (mz, rt, rtmin, rtmax, intensity) with head() output showing feature IDs like F1, F2 and plausible m/z and retention-time-in-seconds values
- Custom csv tables parse successfully only when columns follow the strict order m/z, rt, rtmin, rtmax, then per-sample intensities; misordered or extra columns cause ISFrag to fail reading the file
- Level 1 ISF recognitions should be essentially complete (article benchmark: 100% correct recognition of level 1 features; over 80% for level 2 on metabolite standards)
- get.ISFrag.results() returns a results object sufficient for both exporting the ISF result feature table and calling plot.tree.all()/plot.tree.single() without errors
- In the relationship tree, each fragment feature attaches to a plausible precursor with the precursor retained as the root and level 2/1 fragments as children

## Limitations

- All retention-time columns in custom csv feature tables must be in seconds, not minutes; unit mismatch will corrupt relationship detection
- CAMERA adduct and isotope annotation can only be used for XCMS-only ISFrag analysis
- ISFrag requires R version 4.0.0 or above, and mzXML files must be placed in a folder containing no other irrelevant mzXML files

## Evidence
