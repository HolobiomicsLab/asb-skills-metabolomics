---
name: false-annotation-quantification-omics-scale
description: Use when you have untargeted metabolomics feature tables (mzXML from
  DDA/fullscan plus DDA MS2) and need to identify which features are in-source fragments
  that could be falsely annotated as metabolites, then quantify their prevalence.
when_to_use_negative:
- Your goal is only adduct/isotope annotation with CAMERA — README states CAMERA adduct
  and isotope annotation can only be used for XCMS ONLY ISFrag analysis, and the feature
  table must come from XCMS.
- Your input is raw spectra without a feature table or mzXML files — ISFrag operates
  on MS1 feature tables (XCMS or custom csv) plus DDA mzXML, not arbitrary raw formats.
- Your custom csv feature table lacks the required column order (m/z, rt, rtmin, rtmax
  in seconds, then intensities) — custom.featuretable() will not read it correctly.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ISFrag
  role: R package that performs ISF recognition (levels 1–3), annotation, results
    summarization, and export
  repo: https://github.com/HuanLab/ISFrag
- name: xcms
  role: Extracts MS1 features from mzXML files via XCMS.featuretable(); required for
    CAMERA adduct/isotope annotation
- name: MSnbase
  role: Provides the MSnbase object output alongside the feature table from feature
    extraction
- name: MS-DIAL
  role: Source of custom csv feature tables consumed by custom.featuretable() as an
    alternative to XCMS extraction
- name: devtools
  role: Installs ISFrag from GitHub via install_github("HuanLab/ISFrag")
- name: CAMERA
  role: Optional adduct and isotope annotation, only for XCMS-only ISFrag analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:d1dc11afc0fdde5987dac633dddb42c19bcfc084ad0dad03820dc9457427b730
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag
  tools:
  - CAMERA
  - ISFrag
  - MS-DIAL
  - MSnbase
  - R
  - RStudio
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# false-annotation-quantification-omics-scale

## Summary

Applies the ISFrag R package to untargeted LC-MS/MS metabolomics data (e.g., NIST plasma) to flag in-source fragment (ISF) features that could cause false metabolite annotation, and quantifies their number and proportion relative to the total feature count at the omics scale.

## When to use

Use when you have untargeted metabolomics feature tables (mzXML from DDA/fullscan plus DDA MS2) and need to identify which features are in-source fragments that could be falsely annotated as metabolites, then quantify their prevalence.

## When NOT to use

- Your goal is only adduct/isotope annotation with CAMERA — README states CAMERA adduct and isotope annotation can only be used for XCMS ONLY ISFrag analysis, and the feature table must come from XCMS.
- Your input is raw spectra without a feature table or mzXML files — ISFrag operates on MS1 feature tables (XCMS or custom csv) plus DDA mzXML, not arbitrary raw formats.
- Your custom csv feature table lacks the required column order (m/z, rt, rtmin, rtmax in seconds, then intensities) — custom.featuretable() will not read it correctly.

## Inputs

- mzXML files from fullscan/DDA analyses (MS1 feature extraction input, placed in a dedicated folder)
- mzXML files from DDA analyses (MS2 assignment input)
- XCMS-extracted feature table or custom csv feature table from MS-DIAL/MZmine2 (m/z, rt, rtmin, rtmax in seconds, sample intensity columns)
- Standard spectral library in msp format (e.g., MoNA-export-LC-MS-MS_Negative_Mode.msp)

## Outputs

- Complete annotated feature table with ISF_level, Annotation, and DPscore columns for every feature
- Count and proportion of features flagged as ISF relative to total feature count (omics-scale false-annotation estimate)
- Optional ISF precursor-fragment relationship dataframe / ISF relationship tree

## How to apply

Install ISFrag from GitHub (HuanLab/ISFrag) with devtools (R >= 4.0.0). Build the MS1 feature table either with XCMS.featuretable() (e.g., peakwidth = c(5,20)) or read an MS-DIAL/MZmine2-derived csv via custom.featuretable() (columns: m/z, rt, rtmin, rtmax in seconds, then per-sample intensities). Assign MS2 spectra from DDA mzXML files with ms2.assignment() and annotate features against a standard msp library (e.g., MoNA-export-LC-MS-MS_Negative_Mode.msp) with feature.annotation() (dp = 0.1). Identify ISFs in order from level 3 to level 1 using find.level3(), find.level2(), and find.level1(); CAMERA adduct/isotope annotation is only valid for XCMS-only analyses. Summarize with get.ISFrag.results() and export the annotated table with export.ISFrag.results(); the output carries ISF_level/Annotation/DPscore columns. Finally, count features flagged as ISF and compute their proportion of the total feature count — the fraction of ISF features is the estimate of potential false-annotation sources at the omics scale.

## Related tools

- **ISFrag** (R package that performs ISF recognition (levels 1–3), annotation, results summarization, and export) — https://github.com/HuanLab/ISFrag
- **xcms** (Extracts MS1 features from mzXML files via XCMS.featuretable(); required for CAMERA adduct/isotope annotation)
- **MSnbase** (Provides the MSnbase object output alongside the feature table from feature extraction)
- **MS-DIAL** (Source of custom csv feature tables consumed by custom.featuretable() as an alternative to XCMS extraction)
- **devtools** (Installs ISFrag from GitHub via install_github("HuanLab/ISFrag"))
- **CAMERA** (Optional adduct and isotope annotation, only for XCMS-only ISFrag analysis)

## Examples

```
library(devtools); install_github("HuanLab/ISFrag"); library(ISFrag); xcmsFT <- XCMS.featuretable(MS1directory, type="single", peakwidth=c(5,20))
```

## Evaluation signals

- The exported annotated feature table exists and contains ISF_level, Annotation, and DPscore columns for every feature.
- Level 1 ISF recognition is complete (100% on metabolite standards) and level 2 recognition is above 80%, matching the article's reported performance.
- The ISF proportion is computed as (features flagged as ISF) / (total features) and is reported alongside raw counts.
- Multi-sample custom feature tables contain one intensity column per sample; single-sample tables have exactly m/z, rt, rtmin, rtmax, Intensity.
- MS2 assignment and annotation outputs are non-empty, with dp = 0.1 used for library matching against the msp file.

## Limitations

- Increasing collision energies increases the number of real metabolic features and ISF features, but the percentage of ISF features might not necessarily increase.
- Level 2 ISF recognition is above 80% but not complete (100%) even on metabolite standards, so some level 2 ISFs may remain unflagged.
- CAMERA adduct and isotope annotation can only be used for XCMS-only ISFrag analysis.
- Custom csv feature tables must have retention-time columns in seconds and fixed column order, or custom.featuretable() fails.

## Evidence

- [abstract] omics-scale ISF quantification: "Tested using metabolite standards, we achieved a 100% correct recognition of level 1 ISF features and over 80% correct recognition for level 2 ISF features."
- [intro] package purpose: "an R package for identifying and annotating in-source fragments in LCMS metabolite feature table"
- [methods] workflow ordering: "In-source fragments are identified from Level 3 to Level 1 fragments through functions `find.leve3()`"
- [methods] results export: "Either the complete feature table with ISF relationship annotated in additional columns, or a detailed ISF precursor-fragment relationship dataframe"
- [readme] installation: "if (!requireNamespace("ISFrag", quietly = TRUE)){
  install_github("HuanLab/ISFrag")
}"
- [readme] feature extraction parameters: "xcmsFT <- XCMS.featuretable(MS1directory = MS1directory, type = "single", peakwidth = c(5,20))"
- [readme] custom table format requirement: "it must contain only columns in the following order:
m/z, retention time, min retention time, max retention time, followed by
an additional column containing the intensities of features detected"
- [readme] CAMERA scope restriction: "Note: CAMERA adduct and isotope annotation
can only be used for `XCMS` ONLY `ISFrag` analysis."
