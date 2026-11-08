---
name: cross-validation-spectral-benchmarking
description: Use when benchmarking an in silico structure annotation workflow's confidence
  scoring against known ground-truth reference spectra, comparing merged vs single
  collision-energy MS/MS performance, or reproducing hop-plot/ROC evaluations.
when_to_use_negative:
- Evaluating annotations on biological samples with no ground truth — the article
  notes evaluations were carried out using reference measurements because correct
  answers for biological datasets are unknown
- Spectral library de-replication or sample-level annotation of real LC–MS/MS runs
  rather than benchmarking against known reference structures
- Datasets lacking all three collision energies per compound, or where spectra contain
  at most one peak at some collision energy/noise level (such compounds must be discarded
  per the protocol)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: COSMIC
  role: Computes the confidence score (Platt probabilities from SVM decision values,
    nested-CV parameter C) used for the structure-disjoint evaluation
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Determines molecular formulas de novo from fragmentation and isotope patterns
    at 10 ppm mass accuracy for each collision energy and noise level
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates with default parameters per fold of the ten-fold
    cross-validation
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:084d8e09569b965686a62e7ca6e0effe7ebe729d2866dbd59b271f3557329d70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-validation-spectral-benchmarking

## Summary

Structure-disjoint benchmarking of COSMIC confidence scores via ten-fold cross-validation on Orbitrap reference data (n = 3,721) and an independent QTOF set (n = 3,013), evaluated with hop plots of correct hits versus annotation rate across collision energies (10/20/40 eV, merged) and noise levels.

## When to use

Use when benchmarking an in silico structure annotation workflow's confidence scoring against known ground-truth reference spectra, comparing merged vs single collision-energy MS/MS performance, or reproducing hop-plot/ROC evaluations.

## When NOT to use

- Evaluating annotations on biological samples with no ground truth — the article notes evaluations were carried out using reference measurements because correct answers for biological datasets are unknown
- Spectral library de-replication or sample-level annotation of real LC–MS/MS runs rather than benchmarking against known reference structures
- Datasets lacking all three collision energies per compound, or where spectra contain at most one peak at some collision energy/noise level (such compounds must be discarded per the protocol)

## Inputs

- Combined MassBank/GNPS/NIST 2017 reference spectra for the NIST Orbitrap CV set (3,721 compounds)
- Agilent MassHunter Forensics/Toxicology PCDL spectra for the independent set (3,013 compounds)
- Fragmentation spectra at 10, 20, and 40 eV per compound under no/medium/high noise conditions
- Merged fragmentation spectra per feature (cosine-distance agglomerative clustering)

## Outputs

- Hop plots of correct hits versus annotation rate for 10/20/40 eV and merged spectra, with three noise levels overlaid, per dataset
- Per-condition correct-hit totals at ROC end and annotation rates at the 0.64 threshold (cosmic_hop_plots.csv)

## How to apply

Load the combined MassBank/GNPS/NIST 2017 reference spectra for the NIST Orbitrap cross-validation set (3,721 compounds) and the Agilent MassHunter Forensics/Toxicology PCDL spectra for the independent set (3,013 compounds), retaining only compounds with all three collision energies and discarding any compound whose spectrum at some collision energy/noise level has at most one peak. Process each fragmentation spectrum (10, 20, 40 eV, and merged via cosine-distance agglomerative clustering) with SIRIUS 4 at 10 ppm mass accuracy for de novo molecular formula determination, then rank structure candidates with CSI:FingerID (default parameters). Per fold of the ten-fold cross-validation, map SVM decision values to Platt posterior probabilities and compute the COSMIC confidence score, choosing SVM parameter C from {10^-5,...,10^5} by nested cross-validation. Restrict each candidate list to candidates sharing the molecular formula of the top-scoring hit, then tally correct hits at ROC end per collision energy and merged spectra (10 eV: 1,829; 20 eV: 1,901; 40 eV: 1,765; merged CV-set: 1,948 at ROC end for the merged/independent conditions as reported). Compute annotation rates at the 0.64 confidence-score threshold (~FDR 10%) per condition and render hop plots of correct hits versus annotation rate, overlaying the three noise levels for both datasets, writing metrics to cosmic_hop_plots.csv.

## Related tools

- **COSMIC** (Computes the confidence score (Platt probabilities from SVM decision values, nested-CV parameter C) used for the structure-disjoint evaluation) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (Determines molecular formulas de novo from fragmentation and isotope patterns at 10 ppm mass accuracy for each collision energy and noise level) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates with default parameters per fold of the ten-fold cross-validation) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input reference_spectra.mzML --formula --csi --confidence 10
```

## Evaluation signals

- Correct-hit totals at ROC end match reported values: 1,829 (10 eV), 1,901 (20 eV), 1,765 (40 eV), 1,948 (merged spectra), with merged > 40 eV > other energies ordering
- Annotation rates at the 0.64 confidence-score threshold fall within 34.91–49.15% on the CV set and 41.80–59.98% on the independent set
- Merged spectra and 40-eV spectra yield consistently better annotation rates and separation than single lower energies
- Per-condition metrics for both datasets are present in cosmic_hop_plots.csv alongside the hop plot figures
- Compounds discarded under the at-most-one-peak and all-three-collision-energies filters leave the reported set sizes (3,721 CV; 3,013 independent)

## Limitations

- The 0.64 confidence-score threshold roughly corresponding to FDR 10% is practical guidance only and is no guarantee in either direction
- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- Accurate FDR estimation from small molecule fragmentation spectra is expected to remain highly challenging
- Evaluation relies on reference measurements since correct answers for biological datasets are unknown

## Evidence

- [discussion] hop-plot ROC evaluation: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] confidence threshold: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [discussion] formula restriction: "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [discussion] nested CV: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [discussion] filter steps: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
- [discussion] threshold caveat: "this number might serve as a practical guidance but is clearly no guarantee in either direction"
