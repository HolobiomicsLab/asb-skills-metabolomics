---
name: similarity-score-threshold-classification
description: Use when you must label each peak in an rMSIproc peak matrix as matrix-related
  or not, using isotope-pattern-based similarity scores (S1, S2) and a user threshold
  to separate matrix clusters from endogenous ions.
when_to_use_negative:
- Input is raw profile-mode spectra (imzML) that has not been processed into a centroid-mode
  rMSIproc peak matrix — process first with rMSIproc::ProcessWizard
- You only have a blank/reference sample to subtract — blank subtraction with a 10%
  max-intensity threshold reached only AUC 0.61 with FDR 35.7%, far below rMSIcleanup's
  0.97
- The LDI-promoting material lacks a characteristic, computable isotope pattern for
  the chosen formula
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIcleanup
  role: Computes S1 (spectral) and S2 (intra-cluster morphology) similarity scores
    against theoretical isotope patterns (enviPat) and classifies peaks at a user
    threshold; also runs overlapping-peak detection on low-score clusters
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Generates the centroid-mode peak matrix (.zip) and processed dataset (.tar)
    that annotate_matrix consumes; loads/stores peak matrices
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed MSI data (.tar) required as the full-dataset argument
    for annotate_matrix and report generation
  repo: https://github.com/prafols/rMSI
- name: enviPat
  role: Computes theoretical isotope patterns of the matrix formula used for spectral
    similarity matching
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:a9999176a817c9fcfb28fd1f24c477ef285ab14f0b6ed93376a63e195f97c37d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - enviPat
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# similarity-score-threshold-classification

## Summary

Classifies MSI m/z peaks as matrix-related (e.g., Agn+ silver clusters) vs. endogenous by thresholding per-cluster similarity scores: spectral similarity S1 and intra-cluster morphology similarity S2 computed by rMSIcleanup against theoretical isotope patterns.

## When to use

Use when you must label each peak in an rMSIproc peak matrix as matrix-related or not, using isotope-pattern-based similarity scores (S1, S2) and a user threshold to separate matrix clusters from endogenous ions.

## When NOT to use

- Input is raw profile-mode spectra (imzML) that has not been processed into a centroid-mode rMSIproc peak matrix — process first with rMSIproc::ProcessWizard
- You only have a blank/reference sample to subtract — blank subtraction with a 10% max-intensity threshold reached only AUC 0.61 with FDR 35.7%, far below rMSIcleanup's 0.97
- The LDI-promoting material lacks a characteristic, computable isotope pattern for the chosen formula

## Inputs

- rMSIproc peak matrix (.zip) loaded with rMSIproc::LoadPeakMatrix
- Processed MSI dataset in rMSI format (.tar) loaded with rMSI::LoadMsiData
- Matrix/LDI-material formula string (e.g., 'Ag1')
- Optional: possible/not-possible silver-related peak validation list for accuracy assessment

## Outputs

- Per-peak annotation results with S1 and S2 similarity scores and matrix-related classification (results object from annotate_matrix)
- Per-peak classification and performance summary (e.g., rMSIcleanup_annotations.csv)
- Visual justification PDF report (rMSIcleanup::generate_pdf_report)

## How to apply

Run rMSIcleanup::annotate_matrix on the peak matrix with the LDI-promoting material formula (e.g., 'Ag1'); it computes theoretical isotope patterns with enviPat, matches them to experimental peaks within a mass tolerance of 4 data-points/scans, and returns S1 (spectral similarity) and S2 (intra-cluster morphology similarity) per cluster. Use S1 as the primary classifier (AUC 0.97 on the silver validation list) and S2 as secondary (AUC 0.91); S1·S2 performed no better than S1 alone. Choose the threshold in the score gap between the two classes — in the AgLDI validation, scores between 0.5 and 0.7 separated silver-cluster positives from negatives (Ag salts/adducts such as AgCl, AgBr, AgNO3, and CnHm+Ag wax adducts). Clusters falling below the threshold additionally pass through the overlapping-peak-detection routine (bisecting k-means on spatial distributions) so overlapped or isobaric endogenous ions are re-tagged and not misclassified. Record per-peak classifications and evaluate against a possible/not-possible peak validation list to confirm the separation.

## Related tools

- **rMSIcleanup** (Computes S1 (spectral) and S2 (intra-cluster morphology) similarity scores against theoretical isotope patterns (enviPat) and classifies peaks at a user threshold; also runs overlapping-peak detection on low-score clusters) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Generates the centroid-mode peak matrix (.zip) and processed dataset (.tar) that annotate_matrix consumes; loads/stores peak matrices) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed MSI data (.tar) required as the full-dataset argument for annotate_matrix and report generation) — https://github.com/prafols/rMSI
- **enviPat** (Computes theoretical isotope patterns of the matrix formula used for spectral similarity matching)

## Examples

```
results<-rMSIcleanup::annotate_matrix(pks,"Ag1",full)
```

## Evaluation signals

- Against the silver validation list (positives: Agn+ clusters; negatives: AgCl, AgBr, AgNO3, CnHm+Ag adducts), S1 should reach AUC ≈ 0.97 and S2 ≈ 0.91
- Scores of the two validation classes should show a gap between 0.5 and 0.7 with few misclassifications inside it
- Peaks whose S1/S2 fall below threshold should be re-examined by the bisecting k-means overlapping-peak detection and re-tagged if overlapped with endogenous ions
- The generated PDF report must visually justify each annotation (isotope pattern match and ion-image morphology)
- Downstream PCA on the matrix-removed peak matrix should show enhanced morphological contrast in the first principal components

## Limitations

- Performance depends on a mass tolerance of 4 data-points/scans and on the processing parameters (SNR 5, Savitzky–Golay kernel 7, 5% pixel filter, binning window 6 data-points); different processing may shift score distributions
- Clusters with S1/S2 below the threshold may contain overlapped/isobaric endogenous ions that could be misclassified without the overlapping-peak-detection routine
- Scores were validated on Ag-sputtered AgLDI datasets; transfer to other LDI materials requires re-defining the validation list and re-tuning the threshold

## Evidence

- [methods] S1 best classifier with AUC 0.97; S2 0.91: [evidence span withheld — anchor status: not_found]
- [abstract] Algorithm relies on chemical formula and spatial distribution: "We have developed rMSIcleanup, an open-source R package to annotate and remove matrix-related signals based on its chemical formula and the spatial distribution of its ions."
- [methods] Overlapping-peak detection trigger: "The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user."
- [readme] README annotate_matrix call: "results<-rMSIcleanup::annotate_matrix(pks,"Ag1",full)"
- [methods] Blank subtraction baseline comparison: "The highest AUC of 0.61 reported for Dataset 10 using intensity as the classification metric was well below the reported AUC of 0.97 for rMSIcleanup."
