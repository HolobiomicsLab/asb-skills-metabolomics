---
name: spectrum-disjoint-benchmark-evaluation
description: Use when evaluating structure annotation methods against spectral library
  search baselines on independent tandem MS/MS query data (e.g., medium-noise QTOF),
  requiring a spectrum-disjoint setup and FDR-controlled comparison at 5% FDR.
when_to_use_negative:
- Query compounds whose own reference spectra remain in the library during their evaluation
  — this violates the spectrum-disjoint setup and inflates library-search baseline
  counts
- Biological datasets with no ground truth for correct annotations — evaluations require
  reference measurements with known correct answers
- Claims beyond two-dimensional structure — stereochemistry cannot be established
  from fragmentation spectra, so the benchmark cannot judge stereoisomer correctness
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS 4
  role: De novo molecular formula determination (10 ppm) and fragmentation tree computation
    for each query spectrum, restricting the CSI:FingerID candidate list to the top
    hit's formula
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates under the formula restriction; its training structures
    define the reference library used for the spectrum-disjoint baseline
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Provides confidence scores (kernel density P value estimation plus SVM with
    Platt probability mapping) used to estimate FDR of the top-k hits for the COSMIC
    arm of the comparison
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID 1.2.0
  role: Version used to compute scores for the structure-disjoint evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:f931395db9052dec2820cae17d763c827ff556c4c30ca7f5c49b3420486689ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID 1.2.0
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# spectrum-disjoint-benchmark-evaluation

## Summary

Benchmark evaluation protocol that compares COSMIC in silico structure annotation against spectral library search (normalized dot-product cosine and square-root-intensity variants) at FDR 5%, using a spectrum-disjoint setup on independent QTOF query spectra so each query compound's own reference spectrum is excluded from the library.

## When to use

Use when evaluating structure annotation methods against spectral library search baselines on independent tandem MS/MS query data (e.g., medium-noise QTOF), requiring a spectrum-disjoint setup and FDR-controlled comparison at 5% FDR.

## When NOT to use

- Query compounds whose own reference spectra remain in the library during their evaluation — this violates the spectrum-disjoint setup and inflates library-search baseline counts
- Biological datasets with no ground truth for correct annotations — evaluations require reference measurements with known correct answers
- Claims beyond two-dimensional structure — stereochemistry cannot be established from fragmentation spectra, so the benchmark cannot judge stereoisomer correctness

## Inputs

- Independent QTOF query tandem MS/MS spectra (medium noise level)
- Spectral library built from CSI:FingerID/COSMIC training structures (positive and negative ion mode, via CSI:FingerID web API trainingstructures endpoints)
- SIRIUS 4 fragmentation trees and de novo molecular formula assignments (10 ppm mass accuracy)
- COSMIC confidence scores per structure annotation (kernel density P value + SVM with Platt probability mapping)
- Collision-energy conditions: 20 eV and merged spectra

## Outputs

- fdr5_correct_hits_comparison.csv — table of method x condition (20 eV, merged spectra) x correct hits at FDR 5%
- Per-hit correctness labels for COSMIC and for each library-search scoring variant at exact 5% FDR

## How to apply

Load independent QTOF query tandem MS/MS spectra and build the reference library from CSI:FingerID/COSMIC training structures fetched from the CSI:FingerID web API trainingstructures endpoints (predictor=1 for positive, predictor=2 for negative ion mode). Implement library-search baselines with normalized dot-product cosine similarity and its square-root-intensity-transformed variant. For each query, process spectra with SIRIUS 4 (de novo molecular formula determination at 10 ppm mass accuracy using isotope and fragmentation patterns), rank structure candidates with CSI:FingerID, and restrict the candidate list to candidates sharing the molecular formula of the top-scoring hit. Apply the spectrum-disjoint exclusion — each query compound's own reference spectrum must be removed from the library during its evaluation — then compute COSMIC confidence scores (kernel density P value estimation plus SVM with Platt probability mapping) and estimate FDR as the average posterior error probability of the top-k hits. Count correct annotations at exact 5% FDR for COSMIC and each library-search variant under 20 eV collision energy and under merged spectra, writing a method x condition x correct-hits table to fdr5_correct_hits_comparison.csv.

## Related tools

- **SIRIUS 4** (De novo molecular formula determination (10 ppm) and fragmentation tree computation for each query spectrum, restricting the CSI:FingerID candidate list to the top hit's formula) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates under the formula restriction; its training structures define the reference library used for the spectrum-disjoint baseline) — https://github.com/sirius-ms/sirius
- **COSMIC** (Provides confidence scores (kernel density P value estimation plus SVM with Platt probability mapping) used to estimate FDR of the top-k hits for the COSMIC arm of the comparison) — https://github.com/sirius-ms/sirius
- **CSI:FingerID 1.2.0** (Version used to compute scores for the structure-disjoint evaluation)

## Examples

```
curl "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1" -o trainingstructures_positive.json
```

## Evaluation signals

- Tabulated counts match reported benchmark values: COSMIC 1,415 correct hits vs 52 for spectral library search at 20 eV, and 1,701 vs 1 with merged spectra, at exact 5% FDR
- FDR estimate equals average posterior error probability of the top-k hits, and the 5% FDR threshold is applied exactly (not approximated)
- Each query compound's own reference spectrum is verifiably absent from the library during its evaluation (spectrum-disjoint invariant)
- Library-search baseline counts computed for both normalized dot-product cosine and square-root-intensity-transformed variants under both conditions (20 eV and merged spectra)
- Output CSV schema covers method x condition x correct hits at FDR 5% with all four method-condition combinations populated

## Limitations

- COSMIC was not designed or optimized for de-replication, yet it outperformed spectral library search — the comparison reflects this asymmetry rather than a matched optimization
- The confidence score must not be mistaken for the probability that an annotation is correct; accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging
- The 5% FDR target for the benchmark does not guarantee similar FDR for biological annotations, since evaluations rely on reference measurements while no ground truth exists for biological datasets

## Evidence

- [discussion] Formula restriction rationale for candidate filtering during evaluation: "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [discussion] FDR estimation method underlying the 5% threshold: "we can use the confidence score to estimate the FDR of the top k hits"
- [discussion] Headline outcome of the spectrum-disjoint comparison: "COSMIC clearly outperformed spectral library search for de-replication; this is notable, as COSMIC has not been designed or optimized for this task."
- [readme] Training structures endpoints defining the spectral reference library for positive and negative ion mode: "https://www.csi-fingerid.uni-jena.de/v3.0/api/fingerid/trainingstructures?predictor=1 (training structures for positive ion mode)"
