---
name: surrogate-pooled-qc-feature-coverage-assessment
description: Use when a metabolomics feature matrix includes pooled/surrogate QC injections
  run alongside study samples, and you must determine which features a QC-based drift-correction
  model (e.g. lowess) cannot correct because they are absent from QC injections.
when_to_use_negative:
- When every feature has adequate signal in the pooled QC injections — the coverage
  filter is then a no-op and correction can proceed directly
- When the goal is batch-effect correction of features absent from QCs using statistical
  models (ber, parametric/non-parametric ComBat) that do not depend on QC coverage
  — use dbnorm's Visodbnorm/dbnormSCORE workflow instead
- When input data are raw instrument files (mzML) rather than a post-XCMS feature
  table — feature extraction must precede this assessment
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3272
tools:
- name: LOWESS-Normalization
  role: QC sample-based lowess drift-correction reference implementation whose coverage
    limitation the assessment characterizes; applied to the QC-injected feature subset
- name: dbnorm
  role: R package providing statistical-model alternatives (ber, parametric and non-parametric
    ComBat) that retain QC-undetected features, plus emvd/emvf missing-value estimators
    and dbnormSCORE adjusted-R2 model scoring
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Environment for log2-transforming the feature matrix, compiling the excluded-feature
    list, and running dbnorm/LOWESS-Normalization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:f0dfa5f6e2078309a383ab63dfc3ee32b3e7bce0f2d3ad7419630c3cd1774cb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - LOWESS-Normalization
  - R
  - dbnorm
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# surrogate-pooled-qc-feature-coverage-assessment

## Summary

Identifies metabolic features that are undetected (zero/NA) across all surrogate pooled-tissue QC injections in a log2-transformed untargeted metabolomics matrix, so they can be excluded a priori from QC-based lowess drift correction (as dbnorm's lowess did for xanthosine 5'-monophosphate in SKIPOGH).

## When to use

Use when a metabolomics feature matrix includes pooled/surrogate QC injections run alongside study samples, and you must determine which features a QC-based drift-correction model (e.g. lowess) cannot correct because they are absent from QC injections.

## When NOT to use

- When every feature has adequate signal in the pooled QC injections — the coverage filter is then a no-op and correction can proceed directly
- When the goal is batch-effect correction of features absent from QCs using statistical models (ber, parametric/non-parametric ComBat) that do not depend on QC coverage — use dbnorm's Visodbnorm/dbnormSCORE workflow instead
- When input data are raw instrument files (mzML) rather than a post-XCMS feature table — feature extraction must precede this assessment

## Inputs

- Untargeted metabolomics feature-intensity matrix (.csv) with study samples and pooled-tissue surrogate QC injections, batch levels in the first column
- Sample-type annotation mapping each injection to study sample vs surrogate QC
- Polarity/run-order metadata identifying which QCs were analyzed in each ionization mode

## Outputs

- qc_excluded_features.csv — list of features with zero/NA intensity in all surrogate QC injections (a-priori discarded by QC-based lowess correction)
- qc_lowess_corrected_data.csv — log2-transformed feature table drift-corrected via the QC-based lowess model over the QC-covered subset
- Coverage report distinguishing features detectable only in study samples (e.g. xanthosine 5'-monophosphate) from QC-covered features

## How to apply

Load the untargeted feature matrix (e.g. 232 study samples plus 32 surrogate pooled-tissue QCs injected in random order across batch blocks) and log2-transform each metabolite feature, since dbnorm requires data 'normalized and scaled on the log2'. Scan the QC injection rows only and flag every feature with zero or NA intensity in all surrogate QC injections; these features cannot be fitted by a QC-based lowess drift-correction model and are discarded a priori. Compile these into an explicit excluded-feature list before running LOWESS-Normalization or any QC-based correction, and record which features (such as xanthosine 5'-monophosphate, detected in study samples but missing in the QCs analyzed in the corresponding polarity) fall into this category. Run the QC-based lowess correction on the remaining QC-covered subset, then compare QC-based filtering against statistical-model corrections (ber, parametric/non-parametric ComBat) implemented in dbnorm, which retain QC-undetected features and can be scored with dbnormSCORE's adjusted-R2 metric (0–1).

## Related tools

- **LOWESS-Normalization** (QC sample-based lowess drift-correction reference implementation whose coverage limitation the assessment characterizes; applied to the QC-injected feature subset)
- **dbnorm** (R package providing statistical-model alternatives (ber, parametric and non-parametric ComBat) that retain QC-undetected features, plus emvd/emvf missing-value estimators and dbnormSCORE adjusted-R2 model scoring) — https://github.com/NBDZ/dbnorm
- **R** (Environment for log2-transforming the feature matrix, compiling the excluded-feature list, and running dbnorm/LOWESS-Normalization)

## Examples

```
excluded <- names(which(apply(qc_matrix, 2, function(x) all(is.na(x)) || all(x == 0)))) ; write.csv(data.frame(feature = excluded), "qc_excluded_features.csv", row.names = FALSE)
```

## Evaluation signals

- The excluded-feature list contains exactly those features with zero or NA intensity in every surrogate QC injection — recompute this set independently and confirm the lists match
- xanthosine 5'-monophosphate appears in the excluded list despite being detected in study samples, mirroring the article's finding that lowess 'filtered out xanthosine 5'-monophosphate'
- No feature in the QC-corrected lowess output (qc_lowess_corrected_data.csv) is absent from QC injections
- Cross-check: the same features retained by dbnorm statistical models (ber, parametric/non-parametric ComBat) should be present in the corrected matrix even though lowess discarded them
- dbnormSCORE adjusted-R2 values for corrected features lie in the 0–1 range, with low residual batch variability for well-corrected features (ber/parametric ComBat reached < 0.01 in SKIPOGH)

## Limitations

- QC-based lowess correction systematically discards metabolites below the limit of detection in pooled QCs, so biologically relevant features observed only in study samples (like xanthosine 5'-monophosphate) are lost — a coverage limitation that dbnorm's statistical models do not share
- Residual batch association can persist even after correction: the article reports 'some residual association was observed in lowess- and nonparametric ComBat-corrected data'
- Missing values must be handled before dbnorm correction (via emvd/emvf imputation by lowest detected value), which interacts with coverage filtering decisions

## Evidence

- [results] xanthosine 5'-monophosphate, which was filtered out with lowess: "xanthosine 5′-monophosphate, which was filtered out with lowess"
- [readme] Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features: "Data to be uploaded must be normalized and scaled on the log2 to account for the high abundance features"
- [abstract] some residual association was observed in lowess- and nonparametric Com Bat-corrected data: [evidence span withheld — anchor status: not_found]
