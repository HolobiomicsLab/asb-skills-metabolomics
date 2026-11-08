---
name: cyanogenic-glycoside-peak-area-quantification
description: Use when pooled-plasma LC-MS feature tables (processed with the R-based
  MDRB post-processing technique) are available and you must quantify how much of
  the total plasma peak area is explained by specific identified components, e.g.
  cyanogenic glycosides of CKXR.
when_to_use_negative:
- The peak area table is raw or not MDRB post-processed — relative shares computed
  on unfiltered data will not reflect the exposed-component subset the paper reports
- The target is metabolite identification or prototype–metabolite bridging rather
  than quantifying the exposure share of already-identified features
- Only single-ion-mode data are available, so the positive/negative mode shares cannot
  be computed and compared separately
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging data post-processing technique that
    generates the processed peak area table of identified exposed CKXR components
    used for the quantification
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Language used for the post-processing and for the arithmetic extraction, summation,
    and normalization of cyanogenic glycoside peak areas
- name: PATBS
  role: Non-targeted precise-and-thorough background-substraction technique that identifies
    the metabolite features (e.g. H88/H97, M197/M215) whose peak areas are quantified
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:ff67aa092767dfd37ea9755e99542f2c4ca8d137029babc98b90d027bff3645c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cyanogenic-glycoside-peak-area-quantification

## Summary

Computes the relative exposure share of cyanogenic glycosides (amygdalin and prunasin) from Semen Armeniacae Amarum (CKXR) in pooled plasma by summing feature-level peak areas and dividing by the total peak area, separately per ion mode. In the source study this yielded 43.16% (positive mode) and 53.57% (negative mode) of whole plasma peak areas.

## When to use

Use when pooled-plasma LC-MS feature tables (processed with the R-based MDRB post-processing technique) are available and you must quantify how much of the total plasma peak area is explained by specific identified components, e.g. cyanogenic glycosides of CKXR.

## When NOT to use

- The peak area table is raw or not MDRB post-processed — relative shares computed on unfiltered data will not reflect the exposed-component subset the paper reports
- The target is metabolite identification or prototype–metabolite bridging rather than quantifying the exposure share of already-identified features
- Only single-ion-mode data are available, so the positive/negative mode shares cannot be computed and compared separately

## Inputs

- AUC-pooled plasma peak area table (feature-level peak areas for pooled plasma samples, positive and negative ion modes, MDRB-processed)
- Feature-to-annotation mapping identifying cyanogenic glycoside features (amygdalin: H88 positive / M197 negative; prunasin: H97 positive / M215 negative)

## Outputs

- cyanogenic_glycoside_relative_abundance.csv with columns: ion mode, feature IDs, summed peak area, relative abundance percentage (expected ~43.16% positive, ~53.57% negative)

## How to apply

Load the AUC-pooled plasma peak area table containing feature-level peak areas for pooled plasma samples in both positive and negative ion modes, produced via the R-based MDRB post-processing of PATBS non-targeted data. Extract the rows for the cyanogenic glycoside features: amygdalin (feature H88 in positive mode, M197 in negative mode) and prunasin (feature H97 in positive mode, M215 in negative mode). Sum the peak areas of these features and divide by the total peak area of the pooled plasma dataset, separately for each ion mode, multiplying by 100 to obtain percentage shares. Write the results to cyanogenic_glycoside_relative_abundance.csv with columns for ion mode, feature IDs, summed peak area, and relative abundance percentage. Validate that recomputed shares match the reported 43.16% (positive) and 53.57% (negative) within ±0.5 percentage points; flag deviations for expert review.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging data post-processing technique that generates the processed peak area table of identified exposed CKXR components used for the quantification) — https://github.com/933ZhangDD/MDRB
- **R** (Language used for the post-processing and for the arithmetic extraction, summation, and normalization of cyanogenic glycoside peak areas)
- **PATBS** (Non-targeted precise-and-thorough background-substraction technique that identifies the metabolite features (e.g. H88/H97, M197/M215) whose peak areas are quantified)

## Examples

```
Rscript -e 'd <- read.csv("pooled_plasma_peak_areas.csv"); pos <- sum(d$H88, d$H97); neg <- sum(d$M197, d$M215); c(pos=100*pos/sum(d$positive), neg=100*neg/sum(d$negative))'
```

## Evaluation signals

- Recomputed positive-mode share matches 43.16% within ±0.5 percentage points
- Recomputed negative-mode share matches 53.57% within ±0.5 percentage points
- Output CSV contains the four required columns (ion mode, feature IDs, summed peak area, relative abundance percentage) and one row per ion mode
- Feature IDs used are exactly H88 (amygdalin, positive), H97 (prunasin, positive), M197 (amygdalin, negative), M215 (prunasin, negative)
- Each share equals (summed cyanogenic glycoside peak area / total pooled plasma peak area) × 100 for that ion mode, verifiable by recomputation

## Limitations

- Shares depend on the total pooled plasma peak area denominator, so any change in feature filtering or MDRB post-processing changes the percentage
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the identification strategy underlying the feature assignments
- No software versions or instrument/acquisition parameters are stated for the MDRB/data-processing platform, hindering exact reproduction

## Evidence

- [abstract] R-based post-processing enabling the quantification: "which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique"
- [intro] Exposed CKXR components identified stepwise via MDRB: "the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB"
- [abstract] MDRB combined with PATBS yields the comprehensive in vivo identification strategy: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy"
