---
name: spectral-pairing-annotation-extraction
description: Use when you have MZmine-exported feature quantification CSV (rows filtered
  to features with MS2) and a paired .mgf fragmentation list and need library-level
  annotation scores (cosine + matched peaks) per feature.
when_to_use_negative:
- You only have raw MS1 full-scan data with no MS/MS fragmentation spectra — FBMN
  pairing requires paired .mgf spectra
- Your spectra are in DDA-acquired form only and no SPL-based targeted fragmentation
  was performed; the skill's reference values derive from the RegFilter SPL workflow
- Input is a plain annotated feature table (library hit already assigned); re-running
  spectral pairing would duplicate existing annotation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS2 FBMN workflow
  role: Performs spectral pairing annotation of common-ion nodes against the curated
    in-house/GNPS2 spectral library, returning cosine scores and matched-peak counts
- name: MZmine 2.53
  role: Pre-processes the full-scan and MS/MS data to generate the feature quantification
    .csv and paired .mgf consumed by the FBMN pairing step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:2440da9f692fc0913844d8075869cb0a7cd712b72ed1b639444277f72ee81b72
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - GNPS2
  - GNPS2 FBMN workflow
  - MZmine
  - MZmine 2.53
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# spectral-pairing-annotation-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Searches MZmine-derived MS/MS spectra (feature table + .mgf) against a curated in-house/GNPS2 spectral library via the Feature Based Molecular Network (FBMN) workflow, extracting per-feature library hits, cosine similarity scores, and paired-peak counts. It provides a validation layer to reproduce reported annotation values such as cosine 0.9565 with 25 paired peaks.

## When to use

Use when you have MZmine-exported feature quantification CSV (rows filtered to features with MS2) and a paired .mgf fragmentation list and need library-level annotation scores (cosine + matched peaks) per feature.

## When NOT to use

- You only have raw MS1 full-scan data with no MS/MS fragmentation spectra — FBMN pairing requires paired .mgf spectra
- Your spectra are in DDA-acquired form only and no SPL-based targeted fragmentation was performed; the skill's reference values derive from the RegFilter SPL workflow
- Input is a plain annotated feature table (library hit already assigned); re-running spectral pairing would duplicate existing annotation

## Inputs

- MZmine-exported feature quantification table (.csv, peak area columns, rows filtered to features with MS2)
- Paired fragmentation spectra list (.mgf) produced from SPL-acquired MS/MS data
- Curated in-house spectral library hosted on GNPS2

## Outputs

- Per-feature library annotation with cosine similarity score and paired-peak count
- Results table for the three actinomycete strains (BRA006, BRA010, BRA177) with cosine score and matched-peak count

## How to apply

Load the MZmine 2.53-exported feature quantification .csv (peak-area columns, 'Only with MS2' rows) together with the paired fragmentation .mgf produced from the SPL-acquired data, importing both into the GNPS2 FBMN workflow to pair common-ion nodes against the curated in-house spectral library with adduct consideration [M+H]+ at 15 ppm (0.001 m/z or 15 ppm tolerance used throughout MZmine preprocessing). For each matched node record the library hit, cosine similarity score, and number of paired (matched) peaks, then compile a per-strain results table (BRA006, BRA010, BRA177). The output must reproduce the reported pairs — 0.9565 with 25 peaks for BRA006, 0.3228 with 10 peaks for BRA010, 0.6048 with 13 peaks for BRA177 — so deviations flag library-match mismatches that require expert review rather than silent acceptance.

## Related tools

- **GNPS2 FBMN workflow** (Performs spectral pairing annotation of common-ion nodes against the curated in-house/GNPS2 spectral library, returning cosine scores and matched-peak counts)
- **MZmine 2.53** (Pre-processes the full-scan and MS/MS data to generate the feature quantification .csv and paired .mgf consumed by the FBMN pairing step)

## Examples

```
Upload MZmine feature .csv (Only with MS2) + paired .mgf to GNPS2 FBMN workflow; annotate against curated in-house library ([M+H]+, 15 ppm); extract cosine and paired-peak counts per feature.
```

## Evaluation signals

- The results table must contain a cosine score and paired-peak count row for each of the three strains (BRA006, BRA010, BRA177)
- The computed values must reproduce the reported reference pairs: 0.9565 with 25 peaks for BRA006, 0.3228 with 10 peaks for BRA010, 0.6048 with 13 peaks for BRA177
- Any deviation from the reported cosine/matched-peak pairs triggers expert review of the library match
- The feature .csv used must be the MZmine 'Export/Submit to GNPS-FBMN' export filtered to rows with MS2
- m/z and adduct tolerances ([M+H]+, 15 ppm / 0.001 m/z) must match the MZmine and library-search settings used for the reference values

## Limitations

- Values are library-match dependent: cosine 0.3228 (BRA010) is a weak hit, so results can vary with library curation and are not guaranteed to generalize to other strains
- The pairing uses GNPS2 library; the article also mentions separate spectral pairing in the older GNPS library, so platform choice changes results
- Stochastic DDA acquisition can leave precursor ions unfragmented; this skill only annotates ions that have paired MS/MS spectra, not the full detected feature set

## Evidence

- [results] reported reference pairs: [evidence span withheld — anchor status: not_found]
- [readme] input formats: [evidence span withheld — anchor status: not_found]
