---
name: multivariate-statistical-plotting
description: Use when you have a raw and a QC-RFSC-corrected metabolomics/proteomics
  intensity matrix and need PCA score plots to visually verify tighter QC-sample clustering
  or restored biological group separation (e.g. Figure 3B/3C style).
when_to_use_negative:
- The input is an already-corrected or normalised feature table where drift correction
  and re-plotting QC clustering would be redundant
- The analysis goal is supervised classification of biological groups rather than
  unsupervised QC/batch quality assessment via PCA
- Data lack pooled QC samples injected periodically, so no QC clustering behaviour
  can be evaluated in the score plot
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: statTarget
  role: Provides the statAnalysis function used to compute PCA score plots with Pareto
    scaling and glog transformation on raw and corrected intensities
  repo: github.com/13479776/statTarget
- name: QC-RFSC
  role: Generates the corrected intensity matrix whose PCA score plot is compared
    against the raw-data plot to demonstrate tightened QC clustering
  repo: github.com/13479776/statTarget
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:b4302e17fcd7bb31bd2a8f8b80903e22cf58d0fb01c4b881b1ef24f77669fae1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RFSC
  - statTarget
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multivariate-statistical-plotting

## Summary

Compute and render multivariate PCA score plots (PC1 vs PC2) with statTarget's statAnalysis function, using Pareto scaling and glog transformation, to visualise QC clustering and biological group separation before vs after QC-RFSC signal correction.

## When to use

Use when you have a raw and a QC-RFSC-corrected metabolomics/proteomics intensity matrix and need PCA score plots to visually verify tighter QC-sample clustering or restored biological group separation (e.g. Figure 3B/3C style).

## When NOT to use

- The input is an already-corrected or normalised feature table where drift correction and re-plotting QC clustering would be redundant
- The analysis goal is supervised classification of biological groups rather than unsupervised QC/batch quality assessment via PCA
- Data lack pooled QC samples injected periodically, so no QC clustering behaviour can be evaluated in the score plot

## Inputs

- Raw sample-by-feature intensity matrix (e.g. metabolomics dataset 2, peaks by samples with batch and injection-order meta information)
- QC-RFSC-corrected intensity matrix produced by shiftCor(MLmethod='QCRFSC')
- Sample phenotype/meta file (samPeno) associating each injection with QC or biological group, batch, and injection order

## Outputs

- PCA score coordinates (PC1 vs PC2) per sample for raw and corrected data
- Two PCA score plots (raw vs QC-RFSC-corrected) with distinct colours for QC samples and biological groups and per-sample labels
- Visual assessment of QC cluster variance and biological group separation

## How to apply

Feed the raw and the QC-RFSC-corrected sample intensity matrices separately into statTarget's statAnalysis function with Frule=0.8, normM='NONE', imputeM='KNN', glog=TRUE and scaling='Pareto', which applies the 80% rule for variable retention, KNN imputation of missing values, glog variance stabilisation, and Pareto scaling prior to PCA. Extract the PC1 vs PC2 score coordinates for each sample and render one PCA score plot per data state (raw vs corrected). Colour QC samples distinctly from biological groups and label individual samples (Labels=TRUE) so QC clustering tightness can be judged. The rationale is that correction-induced improvement in precision (e.g. peaks with RSD<30% rising from 52.1% to 90.9% in dataset 2) should manifest as a markedly smaller variance region for QC samples and visible regaining of biological group separation. Accept the output when the corrected-data PCA shows QC samples clustered within a much smaller region than in the raw-data plot, consistent with the reported RSD improvements.

## Related tools

- **statTarget** (Provides the statAnalysis function used to compute PCA score plots with Pareto scaling and glog transformation on raw and corrected intensities) — github.com/13479776/statTarget
- **QC-RFSC** (Generates the corrected intensity matrix whose PCA score plot is compared against the raw-data plot to demonstrate tightened QC clustering) — github.com/13479776/statTarget

## Examples

```
statAnalysis(file, Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto")
```

## Evaluation signals

- The corrected-data PCA score plot shows QC samples clustered within a markedly smaller variance region than in the raw-data plot
- Biological groups that were confounded by drift in the raw plot regain visible separation along PC1/PC2 after correction
- The plotting data derive from matrices where the proportion of peaks with RSD < 30% rose from 52.1% (raw) to 90.9% (corrected) for metabolomics dataset 2
- statAnalysis was run with the paper's parameters (Frule=0.8, normM='NONE', imputeM='KNN', glog=TRUE, scaling='Pareto') and QC samples are coloured/labelled distinctly from biological groups
- Both raw and corrected data were plotted with identical settings so the before/after comparison is directly comparable

## Limitations

- PCA on raw, drift-affected data can show QC samples dispersed across the score plot, so raw and corrected plots must use the same scaling/transformation for a fair comparison
- Requires R (>= 3.3.0); on macOS, XQuartz instead of X11 support should be installed for the statTarget graphical user interface
- Visual QC-cluster tightness is a qualitative signal; it should be corroborated with the quantitative RSD distribution of peaks

## Evidence

- [methods] statTarget used for statistical analysis: [evidence span withheld — anchor status: not_found]
- [readme] README system requirements: [evidence span withheld — anchor status: external_doc]
