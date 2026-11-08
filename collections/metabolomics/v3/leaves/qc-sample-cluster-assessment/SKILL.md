---
name: qc-sample-cluster-assessment
description: Use when a drift-corrected and a raw LC-MS or GC-MS feature table are
  both available and you must visually verify that correction tightened QC clustering
  and restored group separation before accepting the corrected dataset.
when_to_use_negative:
- Input is a single dataset with no paired raw/corrected versions — there is nothing
  to compare for clustering improvement
- The dataset contains no pooled QC injections (e.g. QC-free designs; use statTarget's
  shiftCor_dQC/Combat path instead of QC-based assessment)
- The goal is batch effect quantification by RSD distributions rather than PCA-based
  visual QC clustering
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: statTarget
  role: Provides shiftCor (QC-RFSC correction to generate the corrected matrix) and
    statAnalysis (Pareto scaling, glog, PCA score plot rendering with Labels=TRUE)
  repo: github.com/13479776/statTarget
- name: QC-RFSC
  role: The QC-based random forest signal correction algorithm whose output is being
    assessed via PCA QC clustering
  repo: github.com/13479776/statTarget
- name: random forest
  role: Ensemble learning regression (ntree=500) underlying QC-RFSC drift correction
    being evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:93a519ff064d87756875a4afd0320e85dea906dca3dc517e008b5c63d4373088
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RFSC
  - random forest
  - statTarget
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# qc-sample-cluster-assessment

## Summary

A validation skill for QC-based signal drift correction: build PCA score plots (PC1 vs PC2) from raw and corrected metabolomics/proteomics feature tables and confirm QC samples cluster more tightly and biological group structure is restored after correction (as in Fig. 3B/3C of the statTarget QC-RFSC paper).

## When to use

Use when a drift-corrected and a raw LC-MS or GC-MS feature table are both available and you must visually verify that correction tightened QC clustering and restored group separation before accepting the corrected dataset.

## When NOT to use

- Input is a single dataset with no paired raw/corrected versions — there is nothing to compare for clustering improvement
- The dataset contains no pooled QC injections (e.g. QC-free designs; use statTarget's shiftCor_dQC/Combat path instead of QC-based assessment)
- The goal is batch effect quantification by RSD distributions rather than PCA-based visual QC clustering

## Inputs

- Raw metabolomics/proteomics feature intensity matrix (samples x peaks) with sample meta information including batch and injection order
- QC-RFSC-corrected intensity matrix produced by shiftCor (same dimensions and sample ordering)
- Sample annotation mapping QC vs biological group labels

## Outputs

- PCA score plot (PC1 vs PC2) of raw intensities with QC and biological groups colour-coded
- PCA score plot (PC1 vs PC2) of QC-RFSC-corrected intensities, same styling
- Visual/quantitative judgement that corrected QC cluster variance is reduced and group separation restored

## How to apply

Run the correction (e.g. statTarget's shiftCor with MLmethod='QCRFSC', Frule=0.8, QCspan=0, ntree=500, imputeM='KNN') and keep both raw and corrected intensity matrices. Feed each matrix separately into statAnalysis with Frule=0.8, imputeM='KNN', glog=TRUE and scaling='Pareto' to compute PCA score coordinates for PC1 vs PC2. Render two score plots — raw and corrected — colouring QC samples and biological groups distinctly and labelling samples (Labels=TRUE). Accept the correction when the corrected plot shows QC samples clustered within a markedly smaller variance region and biological groups regaining visible separation, consistent with the paper's precision gain (peaks with RSD<30% rising from 52.1% to 90.9% in metabolomics dataset 2; 43.9% to 86.2% in dataset 1).

## Related tools

- **statTarget** (Provides shiftCor (QC-RFSC correction to generate the corrected matrix) and statAnalysis (Pareto scaling, glog, PCA score plot rendering with Labels=TRUE)) — github.com/13479776/statTarget
- **QC-RFSC** (The QC-based random forest signal correction algorithm whose output is being assessed via PCA QC clustering) — github.com/13479776/statTarget
- **random forest** (Ensemble learning regression (ntree=500) underlying QC-RFSC drift correction being evaluated)

## Examples

```
cor <- shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, ntree = 500, imputeM = "KNN"); statAnalysis(cor, Frule = 0.8, imputeM = "KNN", glog = TRUE, scaling = "Pareto")
```

## Evaluation signals

- Corrected-data PCA shows QC samples clustering within a markedly smaller variance region than in the raw-data PCA
- Biological groups regain visible separation in the corrected PCA score plot
- Independent RSD check agrees with the visual result: proportion of peaks with RSD<30% rises substantially (e.g. 52.1% to 90.9% for metabolomics dataset 2)
- Both raw and corrected plots use identical preprocessing (Frule=0.8, KNN imputation, glog=TRUE, Pareto scaling) so differences are attributable to correction alone
- QC and biological-group samples remain colour-coded and labelled (Labels=TRUE) so cluster tightness can be judged per group

## Limitations

- PCA separation is a visual criterion; the paper corroborates it with quantitative RSD distributions, so plots alone should not be the sole acceptance test
- Requires pooled QC samples injected at regular intervals (e.g. every five, six, or eight real samples in Datasets 1–3) for QC-based correction and assessment
- Performance relative to alternatives differs by method: QC-RLSC and QC-SVR reached only 80.1% and 72.3% peaks within 30% RSDs on metabolomics data, so a less tight QC cluster may reflect the correction algorithm rather than the assessment
- statTarget requires R >= 3.3.0; on macOS, XQuartz (not X11) and RGtk2 2.20.31 are needed for the GUI, which can complicate reproducible rendering

## Evidence
