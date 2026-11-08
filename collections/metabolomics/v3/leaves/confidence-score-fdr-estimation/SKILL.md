---
name: confidence-score-fdr-estimation
description: Use when in-silico structure annotations (e.g. CSI:FingerID candidate
  rankings on LC–MS/MS data) must be ranked by confidence and reported at a controlled
  false discovery rate, especially for structures absent from spectral libraries.
when_to_use_negative:
- Input spectra can already be matched by spectral library search (dot-product cosine
  similarity) — a library search may suffice, though note COSMIC outperformed it even
  for de-replication
- Stereochemistry must be resolved — fragmentation spectra cannot establish stereochemistry,
  only the two-dimensional structure is considered
- The score is needed as the probability that an annotation is correct — this is impossible
  by design of the score
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3895
- http://edamontology.org/topic_2269
tools:
- name: COSMIC
  role: Computes the confidence score (kernel density P value estimation plus SVM
    with Platt posterior probabilities) used for FDR-controlled structure annotation
  repo: github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Determines molecular formulas de novo and computes fragmentation trees, providing
    the input candidates to be scored
  repo: github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates and supplies SVM decision values that are mapped
    to Platt posterior probabilities for scoring
  repo: github.com/sirius-ms/sirius
- name: CSI:FingerID 1.2.0
  role: Provides scores for the structure-disjoint evaluation of CSI:FingerID used
    in the FDR 5% benchmark
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:f83769113159074463b4e8ba25a20d447b78bd576a614b5b4ab0c0833e81954b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID 1.2.0
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3895
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# confidence-score-fdr-estimation

## Summary

COSMIC's confidence-score workflow assigns a statistical confidence to in-silico structure annotations by combining kernel density P value estimation with an SVM whose decision values are mapped to Platt posterior probabilities, enabling FDR-controlled metabolite annotation absent from spectral libraries.

## When to use

Use when in-silico structure annotations (e.g. CSI:FingerID candidate rankings on LC–MS/MS data) must be ranked by confidence and reported at a controlled false discovery rate, especially for structures absent from spectral libraries.

## When NOT to use

- Input spectra can already be matched by spectral library search (dot-product cosine similarity) — a library search may suffice, though note COSMIC outperformed it even for de-replication
- Stereochemistry must be resolved — fragmentation spectra cannot establish stereochemistry, only the two-dimensional structure is considered
- The score is needed as the probability that an annotation is correct — this is impossible by design of the score

## Inputs

- LC–MS/MS runs in .mzML or .mzXML format (centroided, positive ion mode)
- Merged fragmentation spectra per feature (agglomerative clustering on cosine distance)
- De novo molecular formulas from SIRIUS 4 (10 ppm mass accuracy)
- CSI:FingerID structure candidate rankings with SVM decision values
- Structure candidate databases (e.g. biomolecule structure database, combinatorial bile acid database)

## Outputs

- Per-candidate COSMIC confidence scores and Platt posterior probabilities
- Estimated q-values (FDR) for top-k annotations
- High-confidence annotation table filtered at score threshold 0.64 (e.g. 911 annotations from 24,554)
- Ranked novel-annotation tables (e.g. novel_bile_acid_annotations.tsv, fdr5_correct_hits_comparison.csv)

## How to apply

Determine molecular formulas de novo with SIRIUS 4 at 10 ppm mass accuracy, then rank structure candidates with CSI:FingerID against the chosen structure database, restricting candidates to the molecular formula of the top-scoring hit. Train the confidence-score SVM with enforced directionality of features, choosing parameter C from {10^-5, ..., 10^5} by nested cross-validation, and map SVM decision values to posterior probability estimates via Platt probabilities. Compute the COSMIC confidence score (kernel density P value estimation combined with the SVM posterior) and estimate the FDR of the top k hits from the average posterior error probability. Filter annotations for high confidence using the score threshold 0.64, which roughly corresponds to FDR 10% in independent-data evaluation. Optionally cross-reference annotated structures against external databases (e.g. PubChem, HMDB) to retain only truly novel hits before validation.

## Related tools

- **COSMIC** (Computes the confidence score (kernel density P value estimation plus SVM with Platt posterior probabilities) used for FDR-controlled structure annotation) — github.com/sirius-ms/sirius
- **SIRIUS 4** (Determines molecular formulas de novo and computes fragmentation trees, providing the input candidates to be scored) — github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates and supplies SVM decision values that are mapped to Platt posterior probabilities for scoring) — github.com/sirius-ms/sirius
- **CSI:FingerID 1.2.0** (Provides scores for the structure-disjoint evaluation of CSI:FingerID used in the FDR 5% benchmark)

## Evaluation signals

- At FDR 5% on independent QTOF query data, benchmark counts must match: COSMIC 1,415 correct hits vs 52 for spectral library search at 20 eV, and 1,701 vs 1 with merged spectra
- All retained annotations must carry a confidence score >= 0.64 with estimated q-values below 0.1
- Applying the 0.64 threshold to 24,554 structure annotations should yield roughly 911 high-confidence annotations
- FDR is estimated from the average posterior error probability of the top k hits, not from a single hit
- Repository-scale runs should reproduce per-cohort counts (e.g. 436 high-confidence human annotations including 315 without reference MS/MS data; 1,715 novel Orbitrap annotations)

## Limitations

- COSMIC's confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- The 0.64 threshold is practical guidance only, with no guarantee of FDR in either direction on biological datasets
- Accurate FDR estimation from small molecule fragmentation spectra is expected to remain highly challenging
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone
- Evaluations rely on reference measurements because correct answers for biological datasets are unknown

## Evidence

- [abstract] confidence score composition: "a confidence score consisting of kernel density P value estimation and a support vector machine with enforced directionality of features"
- [discussion] FDR threshold calibration: "We have used a threshold of 0.64 for the confidence score, roughly corresponding to FDR 10%"
- [discussion] FDR estimation method: "we can use the confidence score to estimate the FDR of the top k hits"
- [discussion] score interpretation caveat: "COSMIC’s confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score."
- [discussion] SVM hyperparameter selection: "Parameter C ∈ {10−5, 10−4, . . . , 105} of each SVM was chosen by a nested cross-validation."
- [readme] SIRIUS CLI web service integration: "both the
graphical user interface and the command line version of SIRIUS seamlessly integrate the CSI:FingerID, CANOPUS and MSNovelist web services"
