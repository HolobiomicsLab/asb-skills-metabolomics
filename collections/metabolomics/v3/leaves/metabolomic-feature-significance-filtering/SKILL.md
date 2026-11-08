---
name: metabolomic-feature-significance-filtering
description: Use when you have a paired-design LC-MS metabolomic feature table (e.g.
  from XCMS centWave on DDA or DaDIA data) and need to identify statistically significant
  metabolic features for downstream workflow comparison or metabolite annotation.
when_to_use_negative:
- Input is already a significance-annotated feature table or a metabolite list — the
  filtering step would be redundant.
- Data are from an unpaired study design; the paired two-tailed t-test threshold of
  the leukemia plasma study is not valid.
- Data lack method blank samples or MS2 spectra; background-noise filtering and metabolite
  annotation by dot-product matching cannot be performed.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: XCMS
  role: centWave-based feature extraction that produces the feature intensity table
    the significance criteria are applied to
- name: DaDIA.R
  role: Automates feature extraction from DaDIA data and streamlined metabolite annotation,
    providing the input feature table and MS2-based annotations this filtering step
    evaluates
  repo: https://github.com/HuanLab/DaDIA.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:3d748710ddd990f06fdd53db5b6d1c7fd4d050a1f5af2be72e324bdbed2549a1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomic-feature-significance-filtering

## Summary

Applies the leukemia plasma metabolomic significance criteria from the DaDIA study to pick statistically significant metabolic features in XCMS-extracted feature tables, then compares significant-feature counts, MS2 coverage, and annotated endogenous metabolites across acquisition workflows (DDA vs DaDIA).

## When to use

Use when you have a paired-design LC-MS metabolomic feature table (e.g. from XCMS centWave on DDA or DaDIA data) and need to identify statistically significant metabolic features for downstream workflow comparison or metabolite annotation.

## When NOT to use

- Input is already a significance-annotated feature table or a metabolite list — the filtering step would be redundant.
- Data are from an unpaired study design; the paired two-tailed t-test threshold of the leukemia plasma study is not valid.
- Data lack method blank samples or MS2 spectra; background-noise filtering and metabolite annotation by dot-product matching cannot be performed.

## Inputs

- XCMS centWave-extracted feature intensity table from DDA and/or DaDIA mzXML data
- Paired biological sample grouping metadata (e.g. leukemia pre-/post-treatment plasma samples)
- Method blank sample feature intensities
- MS2 spectra for significant features (from DDA-pooled QC or DIA data)
- MS/MS spectral library (e.g. MassBank of North America, MoNA)

## Outputs

- List of statistically significant metabolic features (FC > 1.5 or FC < 0.67 and paired t-test p < 0.05)
- Subset counts of significant features that carry MS2 spectra
- Annotated significant endogenous metabolites with dot product score > 0.7
- Structured comparison table (e.g. leukemia_dda_dadia_comparison.csv) with one row per metric and columns for each acquisition workflow

## How to apply

Extract metabolic features with XCMS centWave-based feature extraction from mzXML data, then remove background noise by filtering features whose intensities are less than 3x the method blank signal. Apply the significance criteria from the leukemia plasma study: fold change (FC) > 1.5 or FC < 0.67 combined with a paired two-tailed t-test p < 0.05. Count total significant features and the subset carrying MS2 spectra, then annotate significant endogenous metabolites by MS2 spectral matching against a library such as MoNA, keeping only matches with a dot product score > 0.7. Validate the output by comparing counts against reference values (e.g. DDA 766 vs DaDIA 993 significant features, DDA 292 vs DaDIA 809 with MS2, DDA 24 vs DaDIA 85 annotated) and by manually inspecting extracted-ion chromatograms (EICs) of novel/rescued features to estimate true- and false-positive rates (leukemia DaDIA-unique features showed 88% true-positive and 5% false-positive).

## Related tools

- **XCMS** (centWave-based feature extraction that produces the feature intensity table the significance criteria are applied to)
- **DaDIA.R** (Automates feature extraction from DaDIA data and streamlined metabolite annotation, providing the input feature table and MS2-based annotations this filtering step evaluates) — https://github.com/HuanLab/DaDIA.git

## Examples

```
Rscript DaDIA.R  # XCMS centWave feature extraction, then filter FC > 1.5 or <0.67 and paired t-test p < 0.05, annotate with MoNA dot product > 0.7
```

## Evaluation signals

- Every reported metric matches the reference values exactly: total significant features (DDA 766, DaDIA 993), significant features with MS2 (DDA 292, DaDIA 809), and annotated significant endogenous metabolites (DDA 24, DaDIA 85).
- DaDIA-unique significant feature count (486) with manual EIC assessment of ~88% true-positive and ~5% false-positive is reproduced.
- Background-noise filter is confirmed: no retained feature has intensity below 3x the method blank signal.
- Every annotated metabolite has a dot product score > 0.7 against the MoNA spectral library.
- Output CSV has one row per metric and columns for each acquisition workflow, with no missing values for DDA or DaDIA.

## Limitations

- Requires a paired sample design (as in the leukemia plasma study); using an independent-samples t-test instead would invalidate the p < 0.05 threshold.
- Significant-feature counts are sensitive to the method-blank threshold (3x blank signal) and to the dot-product score cutoff (0.7); changing these cutoffs changes the reported counts.
- Rescued or DaDIA-unique significant features require manual EIC inspection, and some features may remain undetermined (e.g. the urine study reported 12% undetermined, 17% false-positive).

## Evidence

- [methods] significance filter: [evidence span withheld — anchor status: not_found]
- [results] reference result counts: [evidence span withheld — anchor status: not_found]
