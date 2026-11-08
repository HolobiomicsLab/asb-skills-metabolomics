---
name: lcms-metabolomics-data-preprocessing
description: Use when an LC-MS metabolomics feature table (XCMS/MZmine2/SIEVE/Skyline
  output) with pooled QC samples injected at fixed intervals shows injection-order
  signal drift and needs drift correction before statistical analysis.
when_to_use_negative:
- The dataset has no repeated pooled-QC injections in the run order — use shiftCor_dQC
  with Combat instead, which statTarget provides for QC-free datasets.
- Data are label-free proteomics SRM transition tables rather than metabolomics profiles,
  where the paper's proteomics workflow (Dataset 3, 174 transitions, 69 proteins)
  and its own QC spacing apply.
- Input is already a drift-corrected, quality-filtered feature table; only statAnalysis
  (glog/Pareto, imputation) is needed, not shiftCor.
edam_operation: http://edamontology.org/operation_3636
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: statTarget
  role: R package whose shiftCor function performs the QC-based random forest signal
    correction (QC-RFSC) and QC-RLSC variants, and whose transX, shiftCor_dQC and
    statAnalysis functions cover reformatting, QC-free correction and downstream statistics
  repo: github.com/13479776/statTarget
- name: ProteoWizard
  role: Used to convert LC-MS raw files to mzXML before peak extraction
- name: XCMS
  role: Extracts peak abundances of metabolites from the mzXML data to build the profile
    table
- name: Skyline
  role: Extracts peak abundances (used alongside XCMS) as an alternative source of
    the feature-intensity profile
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:52ac06ead161bf4bcab7c6fba7dba2c418f486b0221005ee64b06aaf19713dba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ProteoWizard
  - QC-RLSC
  - QC-SVR
  - Skyline
  - XCMS
  - shiftCor
  - statTarget
  edam_operation: http://edamontology.org/operation_3636
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# lcms-metabolomics-data-preprocessing

## Summary

Preprocesses and drift-corrects quantitative LC-MS metabolomics feature tables using the statTarget R package (shiftCor with QC-RFSC / QC-RLSC random forest or LOESS correction), turning raw peak-abundance files into precise, batch-corrected data ready for statistical analysis.

## When to use

Use when an LC-MS metabolomics feature table (XCMS/MZmine2/SIEVE/Skyline output) with pooled QC samples injected at fixed intervals shows injection-order signal drift and needs drift correction before statistical analysis.

## When NOT to use

- The dataset has no repeated pooled-QC injections in the run order — use shiftCor_dQC with Combat instead, which statTarget provides for QC-free datasets.
- Data are label-free proteomics SRM transition tables rather than metabolomics profiles, where the paper's proteomics workflow (Dataset 3, 174 transitions, 69 proteins) and its own QC spacing apply.
- Input is already a drift-corrected, quality-filtered feature table; only statAnalysis (glog/Pareto, imputation) is needed, not shiftCor.

## Inputs

- mzXML files converted from vendor LC-MS raw data (ProteoWizard)
- Peak abundance tables from XCMS, MZmine2, SIEVE or Skyline
- statTarget sampleList file (sample names, batch, class, injection order)
- statTarget profile file (feature intensities per sample)
- QC injections every 5–6 aliquoted real samples (Datasets 1–2)

## Outputs

- Drift-corrected feature table at feature level (QC-RFSC/QC-RLSC corrected intensities)
- RSDresult / RSD distribution report of feature CVs across QC samples
- Quality-assessment report identifying peaks that fail the 80% rule
- Corrected dataset formatted for statAnalysis (glog, KNN imputation, Pareto scaling)

## How to apply

First convert vendor MS raw files to mzXML (ProteoWizard) and extract peak abundances with XCMS or Skyline, then use statTarget's transX to reformat those outputs into the required sampleList and profile inputs. Run shiftCor on the QC sample list and profile table with Frule=0.8 (keep features with non-zero values in ≥80% of samples of any group), MLmethod='QCRFSC' (ntree=500, imputeM='KNN'; alternatively QCRLSC with QCspan=0), so that the correction factor Fij is learned per feature and injection order from the QC injections spaced every 5–6 real samples. The QC-RFSC random forest variant was shown to raise the proportion of peaks with RSDs below 30% from 43.9% (raw) to 86.2% on dataset 1, versus 65.5% (QC-RLSC) and 61.9% (QC-SVR), and from 52.1% to 90.9% on dataset 2. Afterwards apply statAnalysis with glog transformation and Pareto scaling for downstream interpretation, and optionally filter low-quality peaks using the quality-assessment criteria after batch integration.

## Related tools

- **statTarget** (R package whose shiftCor function performs the QC-based random forest signal correction (QC-RFSC) and QC-RLSC variants, and whose transX, shiftCor_dQC and statAnalysis functions cover reformatting, QC-free correction and downstream statistics) — github.com/13479776/statTarget
- **ProteoWizard** (Used to convert LC-MS raw files to mzXML before peak extraction)
- **XCMS** (Extracts peak abundances of metabolites from the mzXML data to build the profile table)
- **Skyline** (Extracts peak abundances (used alongside XCMS) as an alternative source of the feature-intensity profile)

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- RSD/CV of every feature across QC injections drops after correction — expect the proportion of peaks with CV<30% to rise to roughly 86% (dataset 1) or 91% (dataset 2) with QC-RFSC, versus ~65%/80% for QC-RLSC.
- Compare the cumulative RSD distribution of corrected vs raw QC features on a single chart; corrected curves must shift toward lower CV bins (e.g. CV<15% reaches 25.1% vs 1.97% raw on dataset 1).
- Feature retention check: features kept after shiftCor must satisfy the Frule=0.8 criterion (non-zero in ≥80% of samples in at least one group).
- Predictive accuracy (R-square) of the drift model on held-out QC injections should show high median and first-quartile values for RF/QRF/Loess fits, as reported for dataset 1.
- The corrected output must be in statTarget's format so statAnalysis(glog, Pareto scaling, KNN imputation) runs without reformatting.

## Limitations

- Correction quality depends on QC injection frequency (every 5 samples in dataset 1, every 6 in dataset 2, every 8 in dataset 3); sparser QC spacing weakens the learned drift model.
- QC-RFSC cannot be applied to datasets without pooled QC samples; statTarget's Combat-based shiftCor_dQC is the QC-free fallback.
- Requires R ≥ 3.3.0; on macOS the GUI needs XQuartz instead of X11, and RGtk2 (2.20.31 with R 3.3.0 was the tested combination) must be installed, which can complicate installation.
- The paper provides funding acknowledgments but no explicit data/code availability statement for the QC-RFSC algorithm beyond the statTarget repository.

## Evidence

- [results] QC-RFSC superiority on dataset 1: [evidence span withheld — anchor status: not_found]
- [readme] System requirements for running the R package: [evidence span withheld — anchor status: external_doc]
