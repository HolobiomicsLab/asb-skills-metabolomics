---
name: silver-isotope-pattern-matching
description: Use when an Ag-sputtered AgLDI mass spectrometry imaging peak matrix
  contains unassigned low-m/z peaks that must be classified as matrix-related (Ag
  clusters/adducts) versus endogenous signals via isotopic-pattern matching.
when_to_use_negative:
- The dataset was not acquired with a silver-sputtered nanolayer (e.g. conventional
  organic MALDI matrix such as CHCA or DHB), since the algorithm relies on the specific
  isotopic pattern of the LDI-promoting material
- Only a peak matrix is available without the processed full spectral data (.tar),
  as annotate_matrix requires both pks and full rMSI data
- Peaks appear in fewer than 5% of pixels or below SNR 5 — such low-quality peaks
  are filtered out during rMSIproc pre-processing and should not be fed to isotope-pattern
  matching
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: rMSIcleanup
  role: Runs annotate_matrix to match enviPat-computed theoretical Ag isotope patterns
    against experimental peaks and assign matrix-related classifications per peak
  repo: https://github.com/gbaquer/rMSIcleanup
- name: enviPat
  role: Computes theoretical isotope fine-structure patterns for Ag clusters and adducts
    used as matching templates
- name: rMSIproc
  role: Provides the centroided peak matrix and processed MSI data that serve as the
    input for pattern matching
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed .tar MSI dataset required alongside the peak matrix
  repo: https://github.com/prafols/rMSI
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:b4f8bc41622cc622eff16864850910980d244d125d6838041845e49854945f95
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
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# silver-isotope-pattern-matching

## Summary

Compute theoretical isotope fine-structure patterns for silver clusters (Ag_n+) and silver-adduct species with enviPat, then match them against experimental AgLDI MSI peak matrices using rMSIcleanup's annotate_matrix to obtain per-peak similarity scores (spectral S1 and intra-cluster morphological S2) for matrix-related peak annotation.

## When to use

Use when an Ag-sputtered AgLDI mass spectrometry imaging peak matrix contains unassigned low-m/z peaks that must be classified as matrix-related (Ag clusters/adducts) versus endogenous signals via isotopic-pattern matching.

## When NOT to use

- The dataset was not acquired with a silver-sputtered nanolayer (e.g. conventional organic MALDI matrix such as CHCA or DHB), since the algorithm relies on the specific isotopic pattern of the LDI-promoting material
- Only a peak matrix is available without the processed full spectral data (.tar), as annotate_matrix requires both pks and full rMSI data
- Peaks appear in fewer than 5% of pixels or below SNR 5 — such low-quality peaks are filtered out during rMSIproc pre-processing and should not be fed to isotope-pattern matching

## Inputs

- rMSIproc peak matrix (.zip) with centroided m/z peaks and pixel intensities
- rMSI processed data (.tar) containing full spectral data in rMSI format
- Matrix formula string (e.g. 'Ag1') defining the theoretical isotope patterns to match

## Outputs

- Per-peak classification results object from annotate_matrix with S1 and S2 similarity scores for each m/z signal
- Visual PDF report justifying each annotation (generate_pdf_report)
- Cleaned peak matrix after remove_matrix (optional downstream step)

## How to apply

Load the rMSIproc peak matrix (.zip) and the processed rMSI data (.tar) with rMSIproc::LoadPeakMatrix and rMSI::LoadMsiData. Compute theoretical isotope patterns with enviPat for the matrix formula supplied to annotate_matrix (here 'Ag1', generating Ag_n+ cluster patterns from the 107Ag/109Ag abundances), and match each pattern against experimental peaks with a mass tolerance of 4 data-points/scans. Two similarity scores are computed per cluster: S1 (spectral similarity) and S2 (intra-cluster morphological similarity); S1 is the better classifier (AUC 0.97 vs 0.91), and the two classes are separable with an S score gap between 0.5 and 0.7. Run the overlapping-peak-detection routine (bisecting k-means on spatial distributions) on clusters whose S1/S2 fall below the user-specified threshold so overlapped or isobaric endogenous ions are not misclassified, then interpret per-peak classifications against the silver possible/not-possible validation list and optionally generate the PDF visual report with generate_pdf_report.

## Related tools

- **rMSIcleanup** (Runs annotate_matrix to match enviPat-computed theoretical Ag isotope patterns against experimental peaks and assign matrix-related classifications per peak) — https://github.com/gbaquer/rMSIcleanup
- **enviPat** (Computes theoretical isotope fine-structure patterns for Ag clusters and adducts used as matching templates)
- **rMSIproc** (Provides the centroided peak matrix and processed MSI data that serve as the input for pattern matching) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed .tar MSI dataset required alongside the peak matrix) — https://github.com/prafols/rMSI

## Examples

```
results <- rMSIcleanup::annotate_matrix(pks, "Ag1", full)
```

## Evaluation signals

- AUC of the spectral similarity score S1 as a classifier against the silver validation list reaches ~0.97 (S2 reaches 0.91)
- Per-peak classifications show a clear S-score gap separating the classes between S = 0.5 and S = 0.7
- Each annotated peak's matched theoretical Ag isotope pattern aligns with the experimental peak within a mass tolerance of 4 data-points/scans
- Clusters with S1/S2 below the user threshold are re-tagged by the overlapping-peak-detection (bisecting k-means) routine rather than misclassified
- PCA on the post-removal peak matrix shows enhanced morphological contrast in the first principal components relative to the pre-removal matrix

## Limitations

- Mass tolerance is fixed at 4 data-points/scans; matching accuracy depends on the spectrometer calibration performed with Ag reference peaks prior to MSI acquisition
- Clusters whose similarity scores fall below the user-specified threshold are additionally screened for overlapping peaks; isobaric endogenous ions could still be misclassified if the overlap detection routine misses them
- Validation depends on the quality of the silver possible/not-possible peak list; entries like Ag salts (AgCl, AgBr, AgNO3) and wax adducts must be correctly curated for fair classification

## Evidence

- [methods] Theoretical isotope pattern matching: "are calculated using the open-source package enviPat [23], a fast and memory-efficient algorithm to compute theoretical isotope patterns."
- [readme] Data loading for annotation: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
- [methods] Mass tolerance parameter: "Peaks within a window of 6 data-points or scans were binned together as the same mass peak."
- [methods] Overlapping peak detection trigger: "The overlapping detection algorithm is only executed in those clusters that report S1 and S2 scores under a threshold specified by the user."
- [results] Classifier performance: [evidence span withheld — anchor status: not_found]
- [intro] Algorithm rationale: "we propose a new algorithm that relies not only on the ion images but also on the chemical information of the LDI promoting material used. The algorithm also incorporates an overlapping peak"
- [methods] Pre-processing quality filter: "Peaks appearing in less than 5% of the pixels were filtered out."
