---
name: matrix-peak-removal-workflow
description: Use when an MSI peak matrix (rMSIproc .zip format) contains redundant,
  non-biologically-relevant matrix-related signals (e.g. Agn+ clusters in silver-assisted
  LDI) that should be annotated and removed before downstream analysis such as PCA
  or spatial metabolomics.
when_to_use_negative:
- Input is raw profile-mode spectra (imzML) rather than an rMSIproc peak matrix —
  first run rMSIproc::ProcessWizard to generate the peak matrix
- The goal is simple blank/off-sample subtraction — rMSIcleanup was designed because
  such approaches ignore ion suppression and matrix adducts with endogenous compounds
  (best blank-subtraction metric only reached AUC 0.61 vs 0.97)
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIcleanup
  role: 'Core package: annotates matrix-related peaks by chemical formula plus spatial
    ion-image distribution, detects overlapping peaks, generates PDF report, and removes
    annotated peaks'
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Provides the peak-matrix format (LoadPeakMatrix/StorePeakMatrix) and the pre-processing
    pipeline (smoothing, calibration, peak-picking, binning) that produces the input
    matrix
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the full processed MSI dataset (.tar) needed as spatial-context input
    to annotate_matrix
  repo: https://github.com/prafols/rMSI
- name: enviPat
  role: Computes theoretical isotope patterns of the matrix formula used to define
    possible silver-related peaks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:917073d0606763162d7399a7a2a4c2e6f73a3c8a406b34feac7d3c4bbcfd8058
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# matrix-peak-removal-workflow

## Summary

Annotates and removes matrix-related peaks (e.g. silver cluster ions in AgLDI MSI) from an rMSIproc peak matrix using rMSIcleanup, which combines the LDI material's chemical formula, theoretical isotope patterns (enviPat), and spatial ion-image distributions, then generates a transparent PDF report justifying each annotation.

## When to use

Use when an MSI peak matrix (rMSIproc .zip format) contains redundant, non-biologically-relevant matrix-related signals (e.g. Agn+ clusters in silver-assisted LDI) that should be annotated and removed before downstream analysis such as PCA or spatial metabolomics.

## When NOT to use

- Input is raw profile-mode spectra (imzML) rather than an rMSIproc peak matrix — first run rMSIproc::ProcessWizard to generate the peak matrix
- The goal is simple blank/off-sample subtraction — rMSIcleanup was designed because such approaches ignore ion suppression and matrix adducts with endogenous compounds (best blank-subtraction metric only reached AUC 0.61 vs 0.97)

## Inputs

- rMSIproc peak matrix (.zip), e.g. Ag_Pancreas_TOF_2015_Dataset2.zip
- rMSI processed MSI dataset (.tar), e.g. Ag_Pancreas_TOF_2015_Dataset2_proc.tar
- Chemical formula of the LDI-promoting material (e.g. 'Ag1')

## Outputs

- Annotation results object from rMSIcleanup::annotate_matrix classifying matrix-related peaks
- PDF visual report justifying each annotation
- Cleaned peak matrix with matrix-related peaks removed (storable via rMSIproc::StorePeakMatrix)

## How to apply

Load the peak matrix with rMSIproc::LoadPeakMatrix and the processed full dataset with rMSI::LoadMsiData (both files are required, e.g. Ag_Pancreas_TOF_2015_Dataset2.zip and .tar from Mendeley Data doi:10.17632/vsk68tjcgh.1). Run rMSIcleanup::annotate_matrix with the matrix chemical formula (e.g. 'Ag1') and the full dataset; the algorithm scores candidate peaks using a spectral similarity score (S1, AUC 0.97 as a classifier) and an intra-cluster morphology similarity score (S2, AUC 0.91), and executes the bisecting k-means based overlapping peak detection only in clusters scoring below a user-specified threshold to avoid misclassifying isobaric or overlapped ions. Optionally call rMSIcleanup::generate_pdf_report to produce a visual report justifying each annotation. Then remove the annotated matrix peaks with rMSIcleanup::remove_matrix and store the cleaned matrix with rMSIproc::StorePeakMatrix; validated parameters for rMSIproc preprocessing include SNR threshold 5, Savitzky–Golay kernel 7, filtering peaks present in <5% of pixels, and binning peaks within a 6 data-point window.

## Related tools

- **rMSIcleanup** (Core package: annotates matrix-related peaks by chemical formula plus spatial ion-image distribution, detects overlapping peaks, generates PDF report, and removes annotated peaks) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Provides the peak-matrix format (LoadPeakMatrix/StorePeakMatrix) and the pre-processing pipeline (smoothing, calibration, peak-picking, binning) that produces the input matrix) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the full processed MSI dataset (.tar) needed as spatial-context input to annotate_matrix) — https://github.com/prafols/rMSI
- **enviPat** (Computes theoretical isotope patterns of the matrix formula used to define possible silver-related peaks)

## Examples

```
pks<-rMSIproc::LoadPeakMatrix("Ag_Pancreas_TOF_2015_Dataset2.zip"); full<-rMSI::LoadMsiData("Ag_Pancreas_TOF_2015_Dataset2_proc.tar"); res<-rMSIcleanup::annotate_matrix(pks,"Ag1",full); pks_clean<-rMSIcleanup::remove_matrix(pks,res)
```

## Evaluation signals

- rMSIcleanup correctly classified m/z signals related to silver clusters (validated across 14 Ag-sputtered MSI datasets from three laboratories)
- Classifier performance benchmarks: spectral similarity score S1 achieved AUC 0.97 and intra-cluster morphology similarity S2 achieved AUC 0.91; S1·S2 matched S1
- After removal, PCA (stats::prcomp, centring only, no scaling or normalization) on the first three PCs should show generalized enhancement of morphological-structure contrast vs the original matrix
- The generated PDF report must exist and justify each annotation transparently
- Cleaned peak matrix must store/load correctly via rMSIproc::StorePeakMatrix/LoadPeakMatrix without matrix-related m/z features

## Limitations

- Simplified approaches (reference-area subtraction, correlation-based methods, off-sample ion classifiers) cannot handle ion suppression, matrix adducts with endogenous compounds, delocalized endogenous molecules, or isobaric/overlapping peaks, and some suffer from the black-box problem
- Blank subtraction with a 10% max-intensity threshold yields poor false discovery rate (35.7%) and false omission rate (11.88%), so it is not an acceptable substitute
- The overlapping detection algorithm only runs on clusters with S1 and S2 scores under a user-specified threshold; mis-tuned thresholds risk misclassification of overlapped or isobaric ions
- Data must be pre-processed into rMSIproc format; rMSIcleanup cannot consume raw imzML directly (use rMSIproc::ProcessWizard)

## Evidence

- [abstract] We have developed rMSIcleanup, an open-source R package to annotate and remove matrix-related signals based on its chemical formula and the spatial distribution of its ions.: "We have developed rMSIcleanup, an open-source R package to annotate and remove matrix-related signals based on its chemical formula and the spatial distribution of its ions."
- [readme] The algorithm incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions.: "The algorithm incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions."
- [readme] The algorithm takes into account the chemical formula and the spatial distribution to determine which ions are matrix-related.: "The algorithm takes into account the chemical formula and the spatial distribution to determine which ions are matrix-related."
- [methods] The Signal-to-Noise Ratio (SNR) threshold was set to 5 and the Savitzky–Golay smoothing had a kernel size of 7.: "The Signal-to-Noise Ratio (SNR) threshold was set to 5 and the Savitzky–Golay smoothing had a kernel size of 7."
- [abstract] it is advisable to annotate and remove matrix-related peaks to reduce the number of redundant and non-biologically-relevant variables in the dataset.: "it is advisable to annotate and remove matrix-related peaks to reduce the number of redundant and non-biologically-relevant variables in the dataset."
