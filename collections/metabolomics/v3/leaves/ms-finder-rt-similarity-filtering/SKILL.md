---
name: ms-finder-rt-similarity-filtering
description: Use when MS/MS spectral matching or in silico MS/MS annotation leaves
  many isomeric structure candidates and predicted LC retention times (from Retip)
  are available to rank or exclude candidates in MS-FINDER 3.30+.
when_to_use_negative:
- Predicting/annotating compounds outside the chemical landscape of the Retip training
  set (e.g., complex high molecular weight lipids such as triacylglycerols or phosphatidylcholines
  were largely excluded from training), where predictions may be more erroneous
- Trying to discriminate closely eluting positional isomers in typical 12 min UPLC
  methods — 1 min prediction errors preclude narrow retention-time search windows
- When no measured retention time exists for the query feature, since the filter compares
  predicted RT against observed RT
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_2258
tools:
- name: Retip
  role: Supplies predicted retention times and the RT.export()/addRT.mona() functions
    that format RT annotations for MS-FINDER
  repo: https://github.com/olobion/Retip
- name: MS-FINDER
  role: Consumes exported .msp RT annotations to rank/filter structure candidates
    by Gaussian RT similarity and RT tolerances (version 3.24+/3.30)
- name: R
  role: Runtime for the Retip R package that produces the exported RT library
  repo: https://github.com/olobion/Retip
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:e5b2cc3a57fb9070c6e52922a1e15cbe7a517da04985c289be9ed69f41236794
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - MS-FINDER
  - R
  - Retip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# MS-FINDER RT similarity filtering

## Summary

Filters and re-ranks MS-FINDER structure/isomer candidates using predicted retention times exported from Retip, applying Gaussian RT similarity scoring and RT tolerance windows (about ±1 min) to remove false-positive annotations.

## When to use

Use when MS/MS spectral matching or in silico MS/MS annotation leaves many isomeric structure candidates and predicted LC retention times (from Retip) are available to rank or exclude candidates in MS-FINDER 3.30+.

## When NOT to use

- Predicting/annotating compounds outside the chemical landscape of the Retip training set (e.g., complex high molecular weight lipids such as triacylglycerols or phosphatidylcholines were largely excluded from training), where predictions may be more erroneous
- Trying to discriminate closely eluting positional isomers in typical 12 min UPLC methods — 1 min prediction errors preclude narrow retention-time search windows
- When no measured retention time exists for the query feature, since the filter compares predicted RT against observed RT

## Inputs

- Predicted retention times for library compounds from Retip (RT.spell/getRT.smile output)
- Source .msp mass search format spectral library file (prepared via prep.mona() and annotated via addRT.mona())
- MS-FINDER compound identification project with isomer candidates (MS-FINDER 3.24+ / 3.30)
- Polarity (pol) of the LC-MS method for RT.export()

## Outputs

- NIST-compatible .msp file carrying predicted RT per compound entry
- MS-FINDER candidate list filtered/re-ranked by RT similarity (reported 68% candidate reduction in isomer search)
- RT tolerance / Gaussian RT similarity scoring configuration for MS-FINDER

## How to apply

First generate predicted retention times with Retip (RT.spell/getRT.smile) and write them back into the .msp mass spectral library via prep.mona()/addRT.mona(), then export with RT.export() using its data, program, and pol (polarity) arguments targeting MS-FINDER. Import the resulting NIST-compatible .msp file into MS-FINDER (version 3.24+/3.30), where Gaussian-based RT similarity scoring and RT tolerance windows are applied alongside accurate mass precursors and MS/MS spectra to penalize candidates whose predicted RT deviates from the observed chromatographic retention. Use a window consistent with model error: Retip's Keras models reached mean absolute errors of 0.78 min for HILIC and 0.57 min for RPLC, and the authors used about ±1 min windows as thresholds for excluding false positives. Expect roughly a 68% reduction in candidate structures when searching all isomers on real samples (mouse blood plasma), but note that such 1 min windows also excluded about 25% of true positives.

## Related tools

- **Retip** (Supplies predicted retention times and the RT.export()/addRT.mona() functions that format RT annotations for MS-FINDER) — https://github.com/olobion/Retip
- **MS-FINDER** (Consumes exported .msp RT annotations to rank/filter structure candidates by Gaussian RT similarity and RT tolerances (version 3.24+/3.30))
- **R** (Runtime for the Retip R package that produces the exported RT library) — https://github.com/olobion/Retip

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Exported .msp is parseable by NIST-compatible MS/MS search software and carries a predicted RT per compound entry
- Candidate structure count in MS-FINDER isomer search drops substantially (article reports 68% reduction on mouse blood plasma)
- RT tolerance window matches model error (about ±1 min; MAE 0.78 min HILIC / 0.57 min RPLC)
- True positive retention acceptable: expect roughly 25% of true positives excluded with 1 min windows
- Polarity argument (pol) of RT.export() matches the acquisition method so RT annotations apply to the correct ion mode

## Limitations

- 1 min windows excluded about 25% of true positives in the authors' models
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; predictions outside the training chemical space may be more erroneous
- None of the current models yield validation MAEs better than ±1 min, so predictions are inadequate for narrow RT search windows with closely eluting compounds
- Whether RT predictions can be adjusted for small changes in chromatographic methods was not tested; to date no chromatographic processing tool actively employs RT predictions for metabolome-wide compound annotations

## Evidence
