---
name: noise-perturbation-robustness-testing
description: Use when you need to test whether a structure-annotation or scoring method
  degrades gracefully under degraded spectral quality — e.g. added noise levels, differing
  collision energies, or merged-versus-single spectra — and want ROC/hop-plot metrics
  as evidence.
when_to_use_negative:
- Input spectra are single-peak or the compound lacks all three collision energies
  — the article discards such compounds entirely, so robustness curves would be incomparable
- You need true FDR guarantees for biological annotations — the article states accurate
  FDR estimation from small-molecule fragmentation spectra is expected to remain highly
  challenging and the 0.64 threshold is practical guidance only
- Input is profiled-mode or negative-ion-mode LC-MS data — such runs were discarded
  in the article's processing (leaving 17,414 LC-MS/MS runs)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS 4
  role: Processes each fragmentation spectrum (per collision energy and noise level)
    for de novo molecular formula determination using isotope and fragmentation patterns
    at 10 ppm mass accuracy
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates with default parameters per fold of the ten-fold
    cross-validation; SVM decision values are mapped to Platt posterior probabilities
    feeding the confidence score
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Confidence-score workflow (kernel density P value estimation plus SVM with
    nested CV over C) whose discriminative power is being stress-tested across noise
    levels and collision energies
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:3a28eeaaf1d1dc72ee026a8ead1f23a59c81b655685c62e70a7d752a069eb134
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# noise-perturbation-robustness-testing

## Summary

Evaluate the robustness of an annotation pipeline (here, SIRIUS 4 → CSI:FingerID → COSMIC confidence score) by re-running it under synthetic noise levels and multiple collision energies (10/20/40 eV plus cosine-distance-merged spectra), then comparing correct-hit totals and annotation rates via hop plots.

## When to use

Use when you need to test whether a structure-annotation or scoring method degrades gracefully under degraded spectral quality — e.g. added noise levels, differing collision energies, or merged-versus-single spectra — and want ROC/hop-plot metrics as evidence.

## When NOT to use

- Input spectra are single-peak or the compound lacks all three collision energies — the article discards such compounds entirely, so robustness curves would be incomparable
- You need true FDR guarantees for biological annotations — the article states accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging and the 0.64 threshold is practical guidance only
- Input is profiled-mode or negative-ion-mode LC-MS data — such runs were discarded in the article's processing (leaving 17,414 LC-MS/MS runs)

## Inputs

- Reference MS/MS spectra per compound at multiple collision energies (10/20/40 eV) in mzML/mzXML format, e.g. combined MassBank/GNPS/NIST 2017 Orbitrap set (n = 3,721)
- Independent-QTOF spectral library, e.g. Agilent MassHunter Forensics/Toxicology PCDL spectra (n = 3,013)
- Structure database for candidate ranking (e.g. HMDB structure database with 113,983 compounds)
- Per-condition noise-level definitions (no/medium/high added noise)

## Outputs

- Hop plots of correct hits versus annotation rate for 10/20/40 eV and merged spectra, overlaid by noise level, for both datasets
- Per-condition metrics table (e.g. cosmic_hop_plots.csv) with ROC-end correct-hit totals and annotation rates at the 0.64 confidence-score threshold

## How to apply

Load reference fragmentation spectra per compound (here MassBank/GNPS/NIST 2017 Orbitrap data, n = 3,721, and Agilent MassHunter Forensics/Toxicology PCDL QTOF data, n = 3,013), retaining only compounds with all three collision energies and discarding any compound whose spectrum at some collision energy/noise level has at most one peak. For each noise level (none/medium/high) and each condition (10, 20, 40 eV, and spectra merged per feature by agglomerative clustering based on cosine distance), run SIRIUS 4 with default parameters at 10 ppm mass accuracy for de novo molecular formula determination, rank structure candidates with CSI:FingerID, map SVM decision values to Platt posterior probabilities, and compute the COSMIC confidence score with nested cross-validation of SVM parameter C over {10^-5,...,10^5}. Restrict each candidate list to candidates sharing the molecular formula of the top-scoring hit, then tally correct hits at ROC end and annotation rates at the 0.64 confidence-score threshold (~FDR 10%). Render hop plots (correct hits vs. annotation rate) per collision energy, overlaying noise levels, for both datasets. Judge success by reproduced totals matching the reported ordering: merged spectra best, usually followed by 40 eV, and annotation rates in the reported ranges (34.91–49.15% CV set; 41.80–59.98% independent set).

## Related tools

- **SIRIUS 4** (Processes each fragmentation spectrum (per collision energy and noise level) for de novo molecular formula determination using isotope and fragmentation patterns at 10 ppm mass accuracy) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates with default parameters per fold of the ten-fold cross-validation; SVM decision values are mapped to Platt posterior probabilities feeding the confidence score) — https://github.com/sirius-ms/sirius
- **COSMIC** (Confidence-score workflow (kernel density P value estimation plus SVM with nested CV over C) whose discriminative power is being stress-tested across noise levels and collision energies) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input spectra.mzML --project noise_robustness --formula --fingerid --cosmic --mass-accuracy 10ppm  # then aggregate ROC-end hits into cosmic_hop_plots.csv per energy/noise level
```

## Evaluation signals

- ROC-end correct-hit totals reproduce the reported values and ordering: merged spectra > 40 eV > other energies (reported 1,901 at 20 eV / 1,829 at 10 eV / 1,765 at 40 eV / 1,948 independent-set merged)
- Annotation rates at the 0.64 confidence-score threshold fall within 34.91–49.15% on the CV set and 41.80–59.98% on the independent set
- Spectra with at most one peak are excluded, and any compound failing this at one collision energy/noise level is discarded from all libraries
- Nested cross-validation picks SVM parameter C from {10^-5,...,10^5} for every condition and fold
- All per-condition metrics are written to cosmic_hop_plots.csv alongside the rendered hop-plot figures

## Limitations

- The confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- The 0.64 threshold (~FDR 10%) is practical guidance only, no guarantee in either direction, and a similar FDR cannot be guaranteed for biological annotations
- Compounds with mass beyond 700 Da were discarded to avoid high running time, and only two-dimensional structures are considered since stereochemistry cannot be established from fragmentation spectra

## Evidence

- [discussion] Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra.: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] Formula determination parameters: "We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm."
- [discussion] Noise/spectral filtering rule: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
- [discussion] Threshold interpretation: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [readme] README tool integration: "SIRIUS integrates a collection of our tools, including CSI:FingerID (with [COSMIC](https://bio.informatik.uni-jena.de/software/cosmic/))"
