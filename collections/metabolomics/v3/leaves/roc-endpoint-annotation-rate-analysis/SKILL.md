---
name: roc-endpoint-annotation-rate-analysis
description: Use when you have confidence-scored structure candidates from multiple
  spectral conditions (e.g. 10/20/40 eV, merged) and need to benchmark how many correct
  hits each condition recovers at a given annotation rate threshold.
when_to_use_negative:
- When input is a feature-abundance table or an already-validated list of compound
  identities rather than candidate rankings with confidence scores.
- When the goal is quantification of abundances rather than benchmarking the correctness/annotation-rate
  trade-off of structure annotation.
- When spectra are in profiled mode or negative ion mode only — the underlying workflow
  discards these (only positive centroided .mzML/.mzXML are processed).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: COSMIC
  role: Confidence-score computation and structure-annotation framework under evaluation;
    its scores drive the hop-plot curves
  repo: https://github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: De novo molecular formula determination from isotope/fragmentation patterns
    at 10 ppm, defining the candidate-restriction step
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates whose decision values feed the confidence score
    used in the hop plots
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:5e1e6c71227cf171c952a00e4e75c6e94e0398fa39061dd397ed0886fa5f6c8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ROC-endpoint annotation-rate analysis

## Summary

Evaluates a confidence-scored in-silico structure annotation workflow (COSMIC/CSI:FingerID) by plotting correct-hit count versus annotation rate across collision energies, noise levels, and datasets, reading each curve's endpoint (ROC end) to recover expected correct-hit totals such as 1,829–1,948 on NIST Orbitrap (n=3,721) and QTOF (n=3,013) sets.

## When to use

Use when you have confidence-scored structure candidates from multiple spectral conditions (e.g. 10/20/40 eV, merged) and need to benchmark how many correct hits each condition recovers at a given annotation rate threshold.

## When NOT to use

- When input is a feature-abundance table or an already-validated list of compound identities rather than candidate rankings with confidence scores.
- When the goal is quantification of abundances rather than benchmarking the correctness/annotation-rate trade-off of structure annotation.
- When spectra are in profiled mode or negative ion mode only — the underlying workflow discards these (only positive centroided .mzML/.mzXML are processed).

## Inputs

- Ten-fold cross-validation set of 3,721 compounds from a combined MassBank/GNPS/NIST 2017 reference (NIST Orbitrap)
- Independent test set of 3,013 compounds from the Agilent MassHunter Forensics/Toxicology PCDL (QTOF)
- Per-compound fragmentation spectra at 10, 20, 40 eV and merged (cosine-distance agglomerative clustering)
- Candidate structure lists ranked by CSI:FingerID (or equivalent) with posterior probabilities
- Molecular formula assignments from SIRIUS 4 at 10 ppm mass accuracy

## Outputs

- Hop plots of correct hits versus annotation rate for each collision energy and noise level, for both datasets
- Per-condition correct-hit totals at ROC end (e.g. 1,829/1,901/1,765/1,948)
- Per-condition annotation rates at the 0.64 confidence-score threshold (34.91–49.15% Orbitrap CV; 41.80–59.98% QTOF)
- CSV of per-condition metrics (cosmic_hop_plots.csv)

## How to apply

Build a ten-fold cross-validation (or independent-set) evaluation where each compound's candidate list is restricted to structures sharing the molecular formula of the top-scoring hit. For each collision energy (10, 20, 40 eV, plus cosine-distance agglomerative-clustering-merged spectra per feature) and noise level, rank candidates with CSI:FingerID, map SVM decision values to Platt posterior probabilities, and compute the COSMIC confidence score with nested cross-validation over SVM parameter C ∈ {10⁻⁵,…,10⁵}. Compute annotation rates at the 0.64 confidence-score threshold (≈ FDR 10%) and tally correct hits at the ROC end for each condition. Expected orderings on the reference data: merged > 40 eV > 10 eV > 20 eV for correct-hit totals (1,948 merged vs. 1,901/1,765/1,829), with annotation rates 34.91–49.15% on the Orbitrap CV set and 41.80–59.98% on the independent QTOF set. The rationale is that hop plots expose the trade-off between how many annotations you accept and how many are correct, so the curve endpoint quantifies the maximum recovery achievable per condition.

## Related tools

- **COSMIC** (Confidence-score computation and structure-annotation framework under evaluation; its scores drive the hop-plot curves) — https://github.com/sirius-ms/sirius
- **SIRIUS 4** (De novo molecular formula determination from isotope/fragmentation patterns at 10 ppm, defining the candidate-restriction step) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates whose decision values feed the confidence score used in the hop plots) — https://github.com/sirius-ms/sirius

## Examples

```
Rscript render_hop_plots.R --input cosmic_hop_plots.csv --output hopplots/ --threshold 0.64
```

## Evaluation signals

- Correct-hit totals at ROC end must match the reported values within tolerance: 1,829 (10 eV), 1,901 (20 eV), 1,765 (40 eV), 1,948 (merged) on the CV set; 1,948 (merged) on the independent set.
- Annotation rates at the 0.64 threshold must fall in the reported ranges (34.91–49.15% Orbitrap CV; 41.80–59.98% QTOF).
- The ordering merged > 40 eV > other energies must hold for correct-hit totals.
- Molecular-formula-restricted candidate lists must be enforced (candidates sharing the top hit's formula) before scoring.
- Per-condition metrics must be written to cosmic_hop_plots.csv alongside the rendered hop-plot figures.

## Limitations

- Confidence score must not be mistaken for the probability that an annotation is correct; impossible by design.
- Threshold 0.64 is practical guidance only, no guarantee in either direction.
- Accurate FDR estimation from small molecule fragmentation spectra expected to remain highly challenging.
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone.
- No ground truth for biological datasets; evaluations rely on reference measurements.

## Evidence

- [methods] Hop plots of correct hits versus annotation rate are the primary evaluation output.: [evidence span withheld — anchor status: not_found]
- [results] Correct-hit totals at ROC end are the primary benchmark values.: [evidence span withheld — anchor status: not_found]
- [discussion] Threshold 0.64 corresponds roughly to FDR 10%.: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [methods] Candidate lists are restricted to those sharing the top hit's molecular formula.: "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [discussion] Merged spectra improve annotation rates and separation.: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [discussion] Confidence score is not a probability of correctness.: [evidence span withheld — anchor status: not_found]
- [methods] SVM decision values are mapped to posterior probabilities via Platt.: "We map decision values to posterior probability estimates using Platt probabilities31."
- [methods] Only compounds with all three collision energies are used.: "we used only those compounds for which all three collision energies are available"
- [methods] Ten-fold cross-validation with nested cross-validation over C.: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [methods] Compounds with at most one peak in any spectrum are discarded.: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
