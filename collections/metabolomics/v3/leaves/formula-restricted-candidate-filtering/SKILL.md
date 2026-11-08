---
name: formula-restricted-candidate-filtering
description: Use when annotating MS/MS spectra with CSI:FingerID/COSMIC after SIRIUS
  4 formula determination and the candidate list must be narrowed to compounds with
  the same molecular formula as the top-scoring hit.
when_to_use_negative:
- When the input is already a spectral library search result (normalized dot-product
  cosine similarity) rather than in silico structure candidates — library search does
  not use formula restriction
- When no reliable de novo molecular formula is available (SIRIUS 4 failed or mass
  accuracy exceeds the 10 ppm setting), since the restriction is keyed to the top
  hit's formula
- When the goal is to compare candidates across different molecular formulas rather
  than rank structures within one formula
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: SIRIUS 4
  role: Determines the de novo molecular formula (isotope + fragmentation patterns,
    10 ppm) that defines the restriction applied to the candidate list
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Ranks structure candidates with default parameters; candidate list is restricted
    to the top hit's molecular formula before scoring
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Assigns confidence scores (kernel density P value estimation plus SVM with
    Platt probability mapping) to the formula-restricted structure annotations
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID 1.2.0
  role: Version used for the structure-disjoint evaluation of scores underlying the
    FDR 5% comparison
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:497e85eaa05b6c0e32dcc3284c34b2dc6ef462697b2fc4a75a72021860a721de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - CSI:FingerID 1.2.0
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# formula-restricted-candidate-filtering

## Summary

Restricts the CSI:FingerID structure candidate list to candidates sharing the de novo molecular formula determined by SIRIUS 4, so that in silico structure ranking and COSMIC confidence scoring operate on a chemically coherent candidate set rather than the full structure database.

## When to use

Use when annotating MS/MS spectra with CSI:FingerID/COSMIC after SIRIUS 4 formula determination and the candidate list must be narrowed to compounds with the same molecular formula as the top-scoring hit.

## When NOT to use

- When the input is already a spectral library search result (normalized dot-product cosine similarity) rather than in silico structure candidates — library search does not use formula restriction
- When no reliable de novo molecular formula is available (SIRIUS 4 failed or mass accuracy exceeds the 10 ppm setting), since the restriction is keyed to the top hit's formula
- When the goal is to compare candidates across different molecular formulas rather than rank structures within one formula

## Inputs

- Independent query tandem MS/MS spectra (e.g., medium-noise QTOF, 20 eV or merged collision energies)
- De novo molecular formula assignments from SIRIUS 4 (10 ppm mass accuracy)
- CSI:FingerID structure candidate rankings (searching a structure database such as the biomolecule structure database or HMDB with 113,983 compounds)
- Reference structure libraries from the CSI:FingerID trainingstructures endpoints (predictor=1 positive ion mode, predictor=2 negative ion mode)

## Outputs

- Formula-restricted structure candidate list per query spectrum
- Ranked structure annotations with COSMIC confidence scores (Platt-mapped posterior probabilities)
- Correct-hit counts at FDR 5% (e.g., comparison table method x condition x correct hits, as in fdr5_correct_hits_comparison.csv)

## How to apply

First process each query spectrum with SIRIUS 4 to determine the molecular formula de novo using isotope and fragmentation patterns with a 10 ppm mass accuracy. Then rank structure candidates with CSI:FingerID under default parameters and restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit); this formula restriction is applied before COSMIC confidence scores (kernel density P value plus SVM with Platt probabilities) are computed. In the FDR 5% benchmark against spectral library search on independent medium-noise QTOF query spectra, this restriction underlies COSMIC's 1,415 correct hits at 20 eV versus 52 for library search, and 1,701 versus 1 with merged spectra. The restriction keeps comparisons against structure-database search fair while preventing lower-formula candidates from diluting the ranking, and pairs naturally with spectrum-disjoint evaluation where each query compound's own reference spectrum is excluded.

## Related tools

- **SIRIUS 4** (Determines the de novo molecular formula (isotope + fragmentation patterns, 10 ppm) that defines the restriction applied to the candidate list) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Ranks structure candidates with default parameters; candidate list is restricted to the top hit's molecular formula before scoring) — https://github.com/sirius-ms/sirius
- **COSMIC** (Assigns confidence scores (kernel density P value estimation plus SVM with Platt probability mapping) to the formula-restricted structure annotations) — https://github.com/sirius-ms/sirius
- **CSI:FingerID 1.2.0** (Version used for the structure-disjoint evaluation of scores underlying the FDR 5% comparison) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --formula --structure --confidence input.mzML --mass-accuracy 10
```

## Evaluation signals

- Tabulated correct hits at exact FDR 5% reproduce the benchmark: COSMIC 1,415 vs 52 for spectral library search at 20 eV, and 1,701 vs 1 with merged spectra
- Every retained structure candidate shares the molecular formula of the top-scoring hit (formula constraint check per query)
- FDR is estimated from the average posterior error probability of the top-k hits; confidence scores are Platt-mapped SVM posteriors, not raw decision values
- Spectrum-disjoint setup holds: each query compound's own reference spectrum is excluded from the library during its evaluation
- Annotation rates and separation should be better with merged spectra, usually followed by 40 eV fragmentation spectra

## Limitations

- COSMIC's confidence score must not be mistaken as the probability that an annotation is correct; this is impossible by design of the score
- Accurate FDR estimation from small-molecule fragmentation spectra is expected to remain highly challenging; the 0.64 threshold (roughly FDR 10%) is practical guidance only, with no guarantee in either direction
- Stereochemistry cannot be established from fragmentation spectra, so only the two-dimensional structure is considered
- Small structure modifications are hard and potentially impossible to tell apart using MS/MS data alone

## Evidence

- [discussion] we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit): "we restrict the candidate list to those candidates with the same molecular formula as the highest-scoring candidate (hit)"
- [discussion] We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm.: "We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm."
- [results] COSMIC searching the biomolecule structure database annotated 1,415 correct hits at 20 eV versus 52 for spectral library search: [evidence span withheld — anchor status: not_found]
- [discussion] We map decision values to posterior probability estimates using Platt probabilities31.: "We map decision values to posterior probability estimates using Platt probabilities31."
- [readme] Training structures for CSI:FingerID's predictors are available through the CSI:FingerID web API: "Training structures for CSI:FingerID's predictors are available through the CSI:FingerID web API"
