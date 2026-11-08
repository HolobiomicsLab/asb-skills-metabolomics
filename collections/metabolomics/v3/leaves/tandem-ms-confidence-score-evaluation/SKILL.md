---
name: tandem-ms-confidence-score-evaluation
description: Use when benchmarking a tandem-MS structure-annotation confidence score
  against known ground truth — reference spectra at 10/20/40 eV, merged spectra, and
  no/medium/high noise — to quantify correct hits versus annotation rate (hop plots)
  and FDR at a score threshold.
when_to_use_negative:
- Annotating biological samples without ground-truth structures — hop-plot correct-hit
  counting requires reference measurements, since 'we do not know the correct answers
  for biological datasets'.
- The goal is spectral-library de-replication rather than benchmarking a confidence
  score against known structures.
- Compounds lack all three collision energies or yield a spectrum with at most one
  peak at some collision energy/noise level — the protocol discards such compounds
  from all libraries.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0157
tools:
- name: SIRIUS 4
  role: Determines molecular formulas de novo (default parameters, 10 ppm mass accuracy)
    from every 10/20/40-eV and merged spectrum per noise level, the prerequisite step
    of the evaluation.
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates (default parameters) within each fold of the ten-fold
    cross-validation; version 1.2.0 was used for the structure-disjoint evaluation
    feeding the confidence score.
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Computes the confidence score (Platt-mapped SVM posterior with nested cross-validation
    of C over {10^-5, ..., 10^5}) that is swept to render the hop plots and set the
    0.64 (~FDR 10%) threshold.
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:73c1fc32f2ccf0f64277c9bb2088781464f7e58ac784e212741e4e51eb956975
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tandem-ms-confidence-score-evaluation

## Summary

Structure-disjoint evaluation of the COSMIC confidence score for tandem-MS structure annotation: ten-fold cross-validation on Orbitrap reference spectra (MassBank/GNPS/NIST 2017, n = 3,721) and an independent Agilent PCDL QTOF set (n = 3,013), rendered as hop plots of correct hits versus annotation rate across 10/20/40-eV and merged spectra under no/medium/high noise.

## When to use

Use when benchmarking a tandem-MS structure-annotation confidence score against known ground truth — reference spectra at 10/20/40 eV, merged spectra, and no/medium/high noise — to quantify correct hits versus annotation rate (hop plots) and FDR at a score threshold.

## When NOT to use

- Annotating biological samples without ground-truth structures — hop-plot correct-hit counting requires reference measurements, since 'we do not know the correct answers for biological datasets'.
- The goal is spectral-library de-replication rather than benchmarking a confidence score against known structures.
- Compounds lack all three collision energies or yield a spectrum with at most one peak at some collision energy/noise level — the protocol discards such compounds from all libraries.

## Inputs

- Reference MS/MS spectra with known structures at 10, 20, and 40 eV from the combined MassBank/GNPS/NIST 2017 database (Orbitrap cross-validation set, n = 3,721 compounds)
- Independent QTOF reference spectra from the Agilent MassHunter Forensics/Toxicology PCDL library (3,243 structures; 3,013 evaluated compounds)
- No/medium/high noise-level variants of every fragmentation spectrum
- Merged fragmentation spectra per feature via agglomerative clustering based on cosine distance
- Structure database for candidate generation (e.g., HMDB, 113,983 compounds, downloaded 8 August 2018)

## Outputs

- Hop plots of correct hits versus annotation rate for 10/20/40 eV and merged spectra, with no/medium/high noise overlays, for both datasets
- End-of-curve correct-hit totals per condition (reported: 1,829 / 1,901 / 1,765 across collision energies and 1,948 for merged spectra)
- Annotation rates at the 0.64 confidence-score threshold (~FDR 10%) per condition
- Per-condition metrics table (cosmic_hop_plots.csv)

## How to apply

Assemble structure-disjoint folds using reference compounds with all three collision energies from the combined MassBank/GNPS/NIST 2017 Orbitrap set (n = 3,721) and the Agilent MassHunter Forensics/Toxicology PCDL QTOF set (n = 3,013), discarding any compound whose spectrum at some collision energy/noise level has at most one peak (and compounds beyond 700 Da, to limit runtime). For each noise level and each spectrum form (10, 20, 40 eV, plus cosine-distance agglomerative-clustering-merged spectra per feature), determine molecular formulas de novo with SIRIUS 4 (default parameters, 10 ppm mass accuracy), rank structure candidates with CSI:FingerID (default parameters) per ten-fold CV fold, map SVM decision values to posterior probabilities via Platt probabilities, and compute the COSMIC confidence score with nested cross-validation of the SVM parameter C over {10^-5, ..., 10^5}. Restrict each candidate list to candidates sharing the molecular formula of the highest-scoring candidate (hit) before tallying correct/incorrect hits. Sweep the confidence-score threshold to render hop plots of correct hits versus annotation rate for 10/20/40 eV and merged spectra, overlaying the three noise levels for both datasets. Validate against the reported end-of-curve correct-hit totals (1,829, 1,901, 1,765 across collision energies; 1,948 for merged spectra), the merged > 40 eV > other energies ordering, and annotation rates at the 0.64 confidence-score threshold (~FDR 10%) within 34.91–49.15% (CV set) and 41.80–59.98% (independent set), writing all per-condition metrics to cosmic_hop_plots.csv alongside the figures.

## Related tools

- **SIRIUS 4** (Determines molecular formulas de novo (default parameters, 10 ppm mass accuracy) from every 10/20/40-eV and merged spectrum per noise level, the prerequisite step of the evaluation.) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates (default parameters) within each fold of the ten-fold cross-validation; version 1.2.0 was used for the structure-disjoint evaluation feeding the confidence score.) — https://github.com/sirius-ms/sirius
- **COSMIC** (Computes the confidence score (Platt-mapped SVM posterior with nested cross-validation of C over {10^-5, ..., 10^5}) that is swept to render the hop plots and set the 0.64 (~FDR 10%) threshold.) — https://github.com/sirius-ms/sirius

## Examples

```
curl "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1"
```

## Evaluation signals

- End-of-curve correct-hit totals match the reported values: 1,829 (10 eV), 1,901 (20 eV), 1,765 (40 eV) and 1,948 (merged spectra) on the cross-validation set.
- Ordering invariant holds: annotation rates and separation are consistently better for merged spectra, usually followed by 40-eV fragmentation spectra.
- Annotation rates at the 0.64 confidence-score threshold fall within 34.91–49.15% (CV set) and 41.80–59.98% (independent QTOF set), roughly corresponding to FDR 10%.
- Exclusion filters applied: only compounds with all three collision energies retained, and any compound with a spectrum of at most one peak at some collision energy/noise level discarded from all libraries.
- Candidate lists restricted to the top-scoring hit's molecular formula before tallying, and all per-condition metrics written to cosmic_hop_plots.csv alongside the hop plot figures.

## Limitations

- COSMIC's confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.
- The 0.64 threshold is practical guidance only (~FDR 10% on independent data); a similar FDR cannot be guaranteed for structure annotations of biological datasets.
- No ground truth exists for biological datasets, so this evaluation design relies on reference measurements.
- Compounds with mass beyond 700 Da were discarded to avoid high running time, bounding the scope of the evaluation.

## Evidence

- [abstract] Here we introduce the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score: "Here we introduce the COSMIC workflow that combines in silico structure database generation and annotation with a confidence score"
- [discussion] We map decision values to posterior probability estimates using Platt probabilities31.: "We map decision values to posterior probability estimates using Platt probabilities31."
- [discussion] Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation.: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [discussion] we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit): "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [discussion] We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [discussion] Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra.: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score.: "COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score."
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]
