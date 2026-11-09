---
name: dot-product-spectral-library-matching
description: Use when annotating MS2 spectra of LC-MS metabolomic features (e.g. DDA,
  DIA(SWATH), or DaDIA data on a Bruker Impact II UHR-QqTOF) by spectral library matching,
  or when binning spectra into high- vs low-quality classes using a dot-product threshold.
when_to_use_negative:
- The analysis goal is feature detection or peak picking — use XCMS centWave-based
  peak picking instead; dot-product matching is applied only after MS2 spectra have
  been extracted from features.
- Input data are already annotated metabolite tables with confidence levels — no further
  spectral matching is needed at that stage.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DaDIA.R
  role: Executes the streamlined metabolite annotation of DaDIA datasets, including
    MS2 spectral matching against the library using dot-product scores.
  repo: https://github.com/Huan Lab/DaDIA.git
- name: MoNA MS spectral library
  role: Reference set of 91 819 standard MS2 spectra against which experimental spectra
    are matched by dot product.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:9c80e04a7d10329151bb28d2fc5fc5603c9c6b36e708bdb89ee5454fdbd7fa25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - MoNA MS spectral library
  - R
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# dot-product-spectral-library-matching

## Summary

Grades experimental MS2 spectra against a standard reference library by computing dot-product (cosine) scores between matched fragment peaks; in the DaDIA urine study, 91 819 MoNA MS2 spectra served as the reference and a score cutoff of 0.7 separated high- from low-quality spectra for metabolite annotation.

## When to use

Use when annotating MS2 spectra of LC-MS metabolomic features (e.g. DDA, DIA(SWATH), or DaDIA data on a Bruker Impact II UHR-QqTOF) by spectral library matching, or when binning spectra into high- vs low-quality classes using a dot-product threshold.

## When NOT to use

- The analysis goal is feature detection or peak picking — use XCMS centWave-based peak picking instead; dot-product matching is applied only after MS2 spectra have been extracted from features.
- Input data are already annotated metabolite tables with confidence levels — no further spectral matching is needed at that stage.

## Inputs

- Experimental MS2 spectra extracted from DDA, DIA(SWATH), or DaDIA mzXML files (e.g. urine study DDA/DIA-(SWATH) data)
- MoNA MS2 spectral library containing 91 819 standard MS2 spectra (downloaded September 20, 2020)
- Dot-product cutoff for quality binning/annotation retention (0.7 as used here)

## Outputs

- Dot-product score per experimental MS2 spectrum against its best library match
- High-/low-quality spectral counts or fractions (e.g. 13% vs 7% high-quality for DaDIA vs DIA(SWATH))
- List of annotated metabolites retaining only matches with dot-product score > 0.7

## How to apply

Match each experimental MS2 spectrum to the MoNA MS spectral library (91 819 standard MS2 spectra, downloaded 2020-09-20) by computing the dot product of the fragment-ion intensity vectors between the experimental and standard spectra. Grade every spectrum this way and apply a cutoff of 0.7: only annotated metabolites with a dot-product score higher than 0.7 are kept for further analysis, and spectra above that threshold are counted as high-quality (scores 90–100 on the reported scale) while those below are low-quality. In the urine study this grading was used to quantify MS2 spectral-quality distributions across workflows (13% high-quality in DaDIA vs 7% in DIA(SWATH); 25% vs 43% low-quality) and to benchmark the 63%/37% DDA/DIA provenance split of MS2 spectra inside DaDIA. Details of the dot-product calculation and MS2 deconvolution are given in the supplementary materials (Text S-6 and S-8), so consult those before reproducing scores exactly.

## Related tools

- **DaDIA.R** (Executes the streamlined metabolite annotation of DaDIA datasets, including MS2 spectral matching against the library using dot-product scores.) — https://github.com/Huan Lab/DaDIA.git
- **MoNA MS spectral library** (Reference set of 91 819 standard MS2 spectra against which experimental spectra are matched by dot product.)

## Examples

```
Rscript DaDIA.R --input mzXML --library MoNA --dot_product_cutoff 0.7
```

## Evaluation signals

- Every experimental MS2 spectrum receives a dot-product score, and the annotated metabolite list contains only entries with score > 0.7.
- Spectral-quality fractions for DaDIA and DIA(SWATH) reproduce the reported values: 13% vs 7% high-quality (scores 90–100) and 25% vs 43% low-quality.
- The DDA/DIA provenance split of MS2 spectra within DaDIA is 63% / 37%; any divergence from these reported figures must be explicitly flagged.
- The library used contains exactly 91 819 standard MS2 spectra from MoNA (downloaded 2020-09-20).
- The 342% feature-increase and 413% MS2-feature-increase of DaDIA over DDA/DIA(SWATH) are consistent with the annotation results produced by this matching step.

## Limitations

- The dot-product cutoff of 0.7 is used for annotation retention and quality binning; matches below this threshold are discarded, so borderline matches may be lost.
- Score reproducibility depends on the exact dot-product and MS2 deconvolution implementations described in supplementary Text S-6 and S-8; reimplementation without consulting them may give different fractions.
- Grading is only as good as the library coverage — metabolites absent from the MoNA library (91 819 spectra, September 2020 snapshot) cannot be annotated this way.

## Evidence
