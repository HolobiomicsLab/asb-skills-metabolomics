---
name: in-source-fragment-identification-lcms
description: Use when an LC-MS/MS metabolomics feature table contains spurious features
  suspected to be in-source fragments, and you need to flag ISF features and their
  precursor-fragment relationships from mzXML/feature-table inputs.
when_to_use_negative:
- The analysis goal is adduct/isotope annotation only — CAMERA adduct and isotope
  annotation can only be used for XCMS-only ISFrag analysis, not custom feature tables
- The data are MS2-only tandem spectra without MS1 feature tables — ISFrag operates
  on MS1 metabolite feature tables, not raw MS2 peak lists
- A fully validated ISF annotation already exists in the feature table, e.g. the exported
  table already contains ISF_level, Num_Level2, and Num_Level1 columns
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ISFrag
  role: Core R package providing XCMS.featuretable(), custom.featuretable(), ms2.assignment(),
    feature.annotation(), find.level3()/find.level2()/find.level1(), get.ISFrag.results(),
    and export functions
  repo: https://github.com/HuanLab/ISFrag
- name: xcms
  role: MS1 feature extraction from mzXML files, including peak alignment and filling
    for multi-sample analyses
  repo: https://rdrr.io/bioc/xcms/man/
- name: MSnbase
  role: Provides the MS spectra object output alongside the dataframe feature table
    from XCMS.featuretable()
- name: devtools
  role: Used to install ISFrag from GitHub via install_github('HuanLab/ISFrag')
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:b8464cf8f27f00f44ce71c16b75e040bcebd087086a14d3cfcfff43e7ab812c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag
  tools:
  - ISFrag
  - MSnbase
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# in-source-fragment-identification-lcms

## Summary

Identifies and annotates in-source fragments (ISFs) in LC-MS metabolite feature tables using the ISFrag R package, which runs a three-level identification cascade (find.level3 → find.level2 → find.level1) over MS1 features with MS2 annotation support.

## When to use

Use when an LC-MS/MS metabolomics feature table contains spurious features suspected to be in-source fragments, and you need to flag ISF features and their precursor-fragment relationships from mzXML/feature-table inputs.

## When NOT to use

- The analysis goal is adduct/isotope annotation only — CAMERA adduct and isotope annotation can only be used for XCMS-only ISFrag analysis, not custom feature tables
- The data are MS2-only tandem spectra without MS1 feature tables — ISFrag operates on MS1 metabolite feature tables, not raw MS2 peak lists
- A fully validated ISF annotation already exists in the feature table, e.g. the exported table already contains ISF_level, Num_Level2, and Num_Level1 columns

## Inputs

- mzXML files from DDA, DIA, or fullscan LC-MS analyses (MS1 folder for feature extraction; separate DDA mzXML folder for MS2 assignment)
- MS1 feature table: XCMS-derived dataframe or custom csv with columns m/z, retention time, min retention time, max retention time (all rt in seconds), plus per-sample intensity columns
- Standard spectral library in msp format (e.g., MoNA-export-LC-MS-MS_Negative_Mode.msp) for feature.annotation()

## Outputs

- Feature table with ISF relationship annotation columns (ISF_level, Num_Level2, Num_Level1) from export.ISFrag.results()
- Detailed per-precursor ISF precursor-fragment relationship dataframes from export.ISFrag.detailed()
- Hierarchical ISF relationship trees from plot.tree.all()
- Summarized level-3/2/1 ISF identification results from get.ISFrag.results()

## How to apply

Build an MS1 feature table with ISFrag's XCMS.featuretable() (peakwidth = c(5,20), type = 'single') on mzXML files, or supply a custom csv (columns: m/z, rt, rtmin, rtmax, per-sample intensities, retention times in seconds) from MS-DIAL/MZmine2 via custom.featuretable(). Assign MS2 spectra to features with ms2.assignment() and annotate against an msp-format standard library such as MoNA-export-LC-MS-MS_Negative_Mode.msp using feature.annotation() with a dot-product score cutoff of 0.1. Then run the ISF cascade in strict order — find.level3() (m/z, retention-time, and intensity-correlation filters), find.level2(), then find.level1() — because fragments are identified from Level 3 to Level 1. Summarize with get.ISFrag.results() and export via export.ISFrag.results(), export.ISFrag.detailed(), and plot.tree.all(). Validated on metabolite standards: 100% correct recognition of level 1 ISF features and over 80% for level 2 ISF features.

## Related tools

- **ISFrag** (Core R package providing XCMS.featuretable(), custom.featuretable(), ms2.assignment(), feature.annotation(), find.level3()/find.level2()/find.level1(), get.ISFrag.results(), and export functions) — https://github.com/HuanLab/ISFrag
- **xcms** (MS1 feature extraction from mzXML files, including peak alignment and filling for multi-sample analyses) — https://rdrr.io/bioc/xcms/man/
- **MSnbase** (Provides the MS spectra object output alongside the dataframe feature table from XCMS.featuretable())
- **devtools** (Used to install ISFrag from GitHub via install_github('HuanLab/ISFrag'))

## Examples

```
library(ISFrag); xcmsFT <- XCMS.featuretable(MS1directory = MS1directory, type = "single", peakwidth = c(5,20)); lvl3 <- find.level3(xcmsFT); lvl2 <- find.level2(lvl3); lvl1 <- find.level1(lvl2); res <- get.ISFrag.results(lvl1)
```

## Evaluation signals

- On metabolite-standard test data, level 1 ISF recognition equals 100% and level 2 recognition exceeds 80% compared to ground truth
- Exported feature table contains the additional Num_Level2 and Num_Level1 columns with ISF_level annotations
- get.ISFrag.results() returns summarized results for all three ISF levels, and plot.tree.all() renders hierarchical ISF trees into the output directory
- The pipeline executed the cascade in the correct order: find.level3() output feeds find.level2(), whose output feeds find.level1()
- Custom csv feature tables pass format checks: only columns m/z, rt, rtmin, rtmax followed by intensity column(s), with all retention-time values in seconds

## Limitations

- CAMERA adduct and isotope annotation can only be used for XCMS-only ISFrag analysis, not with custom feature tables
- Custom feature tables must have retention-time columns in seconds and a fixed column order, or ISFrag cannot read them
- Increasing collision energies can increase the number of real metabolic features and ISF features, but the percentage of ISF features might not necessarily increase
- mzXML files must be placed in a separate folder containing no other irrelevant mzXML files for both feature extraction and MS2 assignment

## Evidence
