---
name: r-package-pipeline-execution
description: Use when an LC-MS feature table must be processed through an R package's
  ordered pipeline functions (e.g. MS1 feature extraction, MS2 annotation, then ISF
  level 3→2→1 identification) rather than a single function call.
when_to_use_negative:
- When only a single downstream operation is needed (e.g. just adduct/isotope annotation),
  rather than the ordered multi-step ISF identification pipeline the package orchestrates.
- When the input is raw instrument data outside the supported mzXML/csv formats and
  no conversion to the required feature table schema (mz, rt, rtmin, rtmax, intensity
  columns) is possible.
- When CAMERA-based adduct and isotope annotation is required but the feature table
  was supplied via custom.featuretable() rather than XCMS — the README states CAMERA
  annotation can only be used for XCMS ONLY ISFrag analysis.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ISFrag
  role: The R package whose ordered pipeline functions (XCMS.featuretable, ms2.assignment,
    feature.annotation, find.level3/2/1, get.ISFrag.results, export.ISFrag.results,
    plot.tree.all) are executed by this skill.
  repo: https://github.com/HuanLab/ISFrag.git
- name: devtools
  role: Installs ISFrag from GitHub via install_github("HuanLab/ISFrag") before the
    pipeline can run.
- name: xcms
  role: Underlies XCMS.featuretable() for MS1 feature extraction from mzXML files,
    including peak alignment and filling for multi-sample analyses.
- name: MSnbase
  role: Provides the MSnbase object returned alongside the feature table by generate.featuretable()/XCMS.featuretable().
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:354c0788ff8d87df24706758272165da4a291068d714f3f6b69c7b89acc79810
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag.git
  tools:
  - ISFrag
  - MSnbase
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# r-package-pipeline-execution

## Summary

Runs a multi-step R scientific package end-to-end (ISFrag: in-source fragment recognition in LC-MS feature tables) from installation via devtools through ordered function calls to exported result tables and trees.

## When to use

Use when an LC-MS feature table must be processed through an R package's ordered pipeline functions (e.g. MS1 feature extraction, MS2 annotation, then ISF level 3→2→1 identification) rather than a single function call.

## When NOT to use

- When only a single downstream operation is needed (e.g. just adduct/isotope annotation), rather than the ordered multi-step ISF identification pipeline the package orchestrates.
- When the input is raw instrument data outside the supported mzXML/csv formats and no conversion to the required feature table schema (mz, rt, rtmin, rtmax, intensity columns) is possible.
- When CAMERA-based adduct and isotope annotation is required but the feature table was supplied via custom.featuretable() rather than XCMS — the README states CAMERA annotation can only be used for XCMS ONLY ISFrag analysis.

## Inputs

- mzXML file(s) from fullscan/DDA (or DIA) LC-MS analyses in a dedicated folder
- MS1 feature table (XCMS-generated dataframe, or custom csv with columns m/z, rt, rtmin, rtmax in seconds plus per-sample intensity columns)
- Standard MS/MS library in msp format (e.g. MoNA-export-LC-MS-MS_Negative_Mode.msp)

## Outputs

- Dataframe-format MS1 feature table (e.g. columns mz, rt, rtmin, rtmax, maxo) plus an MSnbase object
- Feature table with ISF relationship annotations (ISF_level columns, Num_Level2 / Num_Level1) after export.ISFrag.results()
- Detailed ISF precursor-fragment relationship dataframe
- Hierarchical ISF relationship trees rendered by plot.tree.all()

## How to apply

Install the package with devtools::install_github("HuanLab/ISFrag") (R ≥ 4.0.0, RStudio recommended), then load it with library(ISFrag). Build the MS1 feature table either with XCMS.featuretable() on mzXML files (peakwidth = c(5, 20), type = "single") or with custom.featuretable() on a csv whose columns must be m/z, rt, rtmin, rtmax in seconds followed by per-sample intensities. Assign MS2 spectra from DDA mzXML files via ms2.assignment() and annotate features against an msp-format standard library (e.g. MoNA-export-LC-MS-MS_Negative_Mode.msp) with feature.annotation(). Then run the ISF identification functions in strict order — find.level3(), find.level2(), find.level1() — because fragments are identified from Level 3 down to Level 1, and summarize with get.ISFrag.results(). Finally export via export.ISFrag.results() (complete feature table with ISF annotations, or a detailed precursor-fragment dataframe) and plot hierarchical ISF trees with plot.tree.all(). Parameters of individual functions are documented under help("<function name>"); keep each step's output object (dataframe feature table, MSnbase object) as input to the next stage.

## Related tools

- **ISFrag** (The R package whose ordered pipeline functions (XCMS.featuretable, ms2.assignment, feature.annotation, find.level3/2/1, get.ISFrag.results, export.ISFrag.results, plot.tree.all) are executed by this skill.) — https://github.com/HuanLab/ISFrag.git
- **devtools** (Installs ISFrag from GitHub via install_github("HuanLab/ISFrag") before the pipeline can run.)
- **xcms** (Underlies XCMS.featuretable() for MS1 feature extraction from mzXML files, including peak alignment and filling for multi-sample analyses.)
- **MSnbase** (Provides the MSnbase object returned alongside the feature table by generate.featuretable()/XCMS.featuretable().)

## Examples

```
xcmsFT <- XCMS.featuretable(MS1directory = MS1directory, type = "single", peakwidth = c(5,20))
```

## Evaluation signals

- The exported feature table from export.ISFrag.results() carries the ISF relationship annotation columns (Num_Level2 and Num_Level1 / ISF_level) as described in the README's Part 5.
- The pipeline completed with no missing-stage errors: find.level3() ran before find.level2(), which ran before find.level1(), matching the package's required Level 3 → Level 2 → Level 1 ordering.
- The MS1 feature table columns are in the required order (m/z, rt, rtmin, rtmax, intensities) and all retention times are in seconds, otherwise custom.featuretable() will not parse the csv correctly.
- Custom csv feature tables load without column-order errors, and XCMS.featuretable() returns both a dataframe and an MSnbase object.
- Validation against the article's performance benchmark: level 1 ISF recognition should be 100% correct and level 2 recognition over 80% correct on metabolite-standard test data.

## Limitations

- ISFrag requires R version 4.0.0 or above; older R installations cannot install the package.
- CAMERA adduct and isotope annotation works only when the feature table comes from XCMS (XCMS ONLY ISFrag analysis), not from custom feature tables.
- For multi-sample analyses, peak alignment and filling are delegated to XCMS, so behavior depends on XCMS settings.
- The ms2.annotation workflow requires one or multiple DDA mzXML files, and the number of MS2 mzXML files need not match the number of MS1 files; non-DDA MS2 acquisition cannot drive feature.annotation().
- Tested performance (100% correct level 1 ISF recognition, >80% level 2) was demonstrated using metabolite standards, so accuracy on complex unstandardized matrices may vary.

## Evidence
