---
name: overlapping-peak-detection
description: Use when annotating matrix-related peaks (e.g. Agn+ clusters with formula
  'Ag1') in MSI peak matrices and a cluster's similarity scores S1/S2 fall below the
  user threshold, indicating a possibly co-isolated endogenous peak.
when_to_use_negative:
- When peaks are already confirmed pure matrix signals with S1/S2 above the user threshold
  — the routine is only executed for low-scoring clusters
- When the goal is general peak picking/binning of raw spectra rather than re-tagging
  matrix annotations — use rMSIproc preprocessing (SNR 5, 6 data-point binning window,
  5% pixel filter) instead
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: rMSIcleanup
  role: Implements the overlapping peak detection (bisecting k-means on spatial distributions)
    within annotate_matrix and exposes it to remove_matrix and the PDF report
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Generates and loads the centroid-mode peak matrix whose per-cluster spatial
    distributions feed the k-means routine
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed ion-image data used as the spatial features for the bisecting
    k-means clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:3a00a9dd84db83014ed29ff0105fd5e0f55dc6f57b96f76412a1a597bcd67b7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# overlapping-peak-detection

## Summary

A bisecting k-means-based routine inside rMSIcleanup that re-tags m/z peaks whose spatial distributions are mixtures of a matrix-related ion and an endogenous/isobaric ion, preventing misclassification of overlapped or isobaric ions during matrix annotation.

## When to use

Use when annotating matrix-related peaks (e.g. Agn+ clusters with formula 'Ag1') in MSI peak matrices and a cluster's similarity scores S1/S2 fall below the user threshold, indicating a possibly co-isolated endogenous peak.

## When NOT to use

- When peaks are already confirmed pure matrix signals with S1/S2 above the user threshold — the routine is only executed for low-scoring clusters
- When the goal is general peak picking/binning of raw spectra rather than re-tagging matrix annotations — use rMSIproc preprocessing (SNR 5, 6 data-point binning window, 5% pixel filter) instead

## Inputs

- rMSIproc peak matrix loaded with rMSIproc::LoadPeakMatrix (e.g. Ag_Pancreas_TOF_2015_Dataset2.zip)
- Processed rMSI dataset loaded with rMSI::LoadMsiData (e.g. Ag_Pancreas_TOF_2015_Dataset2_proc.tar) supplying ion images for spatial clustering
- Per-cluster S1 (spectral) and S2 (intra-cluster morphological) similarity scores from rMSIcleanup::annotate_matrix with matrix formula 'Ag1' and mass tolerance of 4 data-points/scans

## Outputs

- Re-tagged per-peak annotations in which overlapped/isobaric ions are distinguished from pure matrix-related peaks (rMSIcleanup_annotations.csv)
- Updated results object consumed by rMSIcleanup::generate_pdf_report for visual justification of each annotation

## How to apply

After computing per-cluster similarity scores S1 (spectral) and S2 (intra-cluster morphological) with rMSIcleanup::annotate_matrix against theoretical isotope patterns from enviPat, run the overlapping peak detection only on clusters whose S1 and S2 fall below the user-specified threshold. The routine applies bisecting k-means to the spatial distributions (ion images) of the peaks in the cluster to test whether the observed signal is a pure matrix ion image or a mixture of matrix and an overlapped/isobaric endogenous ion; mixed distributions are re-tagged so the endogenous peak is not removed as matrix-related. This step is what guards the downstream classification against false removal of isobaric biological signals, and its per-peak decisions are justified in the PDF report.

## Related tools

- **rMSIcleanup** (Implements the overlapping peak detection (bisecting k-means on spatial distributions) within annotate_matrix and exposes it to remove_matrix and the PDF report) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Generates and loads the centroid-mode peak matrix whose per-cluster spatial distributions feed the k-means routine) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed ion-image data used as the spatial features for the bisecting k-means clustering)

## Examples

```
results <- rMSIcleanup::annotate_matrix(pks, "Ag1", full); rMSIcleanup::generate_pdf_report(results, pks, full, "test", folder="/home/user/")
```

## Evaluation signals

- Overlapping detection fires only for clusters with S1 and S2 below the user threshold — check that no cluster above threshold was re-tagged
- Re-tagged peaks should show bimodal/mixed spatial distributions under bisecting k-means rather than a single matrix-like ion image
- Against the silver validation list, per-peak classifications should show the reported class separation between S of 0.5 and 0.7, with overall AUC 0.97 for S1
- generate_pdf_report should produce a visual justification for each re-tagged annotation, and remove_matrix should retain (not delete) re-tagged endogenous peaks

## Limitations

- Only executed on low-scoring clusters, so misclassification above threshold is not corrected
- Depends on quality of spatial distributions; delocalized endogenous molecules may confound the k-means separation
- Requires rMSIproc-format data — raw imzML must first be processed via rMSIproc::ProcessWizard

## Evidence

- [methods] The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user.: "The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user."
- [readme] The algorithm incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions.: "The algorithm incorporates an overlapping peak detection feature to prevent misclassification of overlapped or isobaric ions."
- [intro] we propose a new algorithm that relies not only on the ion images but also on the chemical information of the LDI promoting material used. The algorithm also incorporates an overlapping peak: "we propose a new algorithm that relies not only on the ion images but also on the chemical information of the LDI promoting material used. The algorithm also incorporates an overlapping peak"
