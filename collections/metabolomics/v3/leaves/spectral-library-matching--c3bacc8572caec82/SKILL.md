---
name: spectral-library-matching
description: Use when MS/MS spectra (.mgf) paired with a feature quantification table
  (.csv) need library annotations, and each matched node's cosine score and matched-peak
  count must be extracted and compared to reference values.
when_to_use_negative:
- The input feature table contains only MS1 features without MS2 spectra (FBMN export
  must filter rows 'Only with MS2'); use ChemWalker or SIRIUS annotation layers instead
- No curated in-house or public spectral library exists for the compound classes of
  interest; library matching will return no hits and SIRIUS in silico annotation is
  the appropriate fallback
- The goal is de novo formula/structure ranking rather than matching against existing
  spectra; SIRIUS with Q-TOF parameters and adducts [M+H]+, [M+K]+, [M+Na]+ is designed
  for that
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2.53
  role: Preprocesses full scan and MS/MS data separately (m/z tolerance 0.001 m/z
    or 15 ppm) and exports the .csv feature quantification table and .mgf spectra
    list fed to FBMN
- name: GNPS2 (FBMN workflow)
  role: Performs molecular networking and spectral pairing annotation of common-ion
    nodes against the curated in-house/GNPS2 spectral library
- name: GNPS library
  role: Alternative public library used for spectral pairing of SPL-derived spectra
- name: RegFilter
  role: Generates the scheduled precursor list (SPL) whose list-only ions are the
    spectra subsequently matched against the library
  repo: https://github.com/computational-chemical-biology/regression_filter
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:8a58f2d49fd2cc1e42336f8c5259e60f429335b5a44bc0dc1c2052d7324d9fdd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - GNPS
  - GNPS library
  - GNPS2
  - GNPS2 (FBMN workflow)
  - MZmine
  - MZmine 2.53
  - RegFilter
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates fragmentation spectra by spectral pairing of MZmine-exported feature tables (.csv) and MS/MS spectra lists (.mgf) against curated in-house or public GNPS2 libraries via the Feature Based Molecular Network (FBMN) workflow, returning cosine scores and paired-peak counts per feature.

## When to use

Use when MS/MS spectra (.mgf) paired with a feature quantification table (.csv) need library annotations, and each matched node's cosine score and matched-peak count must be extracted and compared to reference values.

## When NOT to use

- The input feature table contains only MS1 features without MS2 spectra (FBMN export must filter rows 'Only with MS2'); use ChemWalker or SIRIUS annotation layers instead
- No curated in-house or public spectral library exists for the compound classes of interest; library matching will return no hits and SIRIUS in silico annotation is the appropriate fallback
- The goal is de novo formula/structure ranking rather than matching against existing spectra; SIRIUS with Q-TOF parameters and adducts [M+H]+, [M+K]+, [M+Na]+ is designed for that

## Inputs

- MZmine-exported feature quantification table (.csv) with peak area columns, rows filtered to features with MS2
- Fragmentation spectra list (.mgf) produced from SPL-acquired MS/MS data pre-processed with MZmine 2.53 (m/z tolerance 0.001 m/z or 15 ppm)
- Curated in-house and/or GNPS2 spectral library

## Outputs

- Per-strain table of library hits with cosine similarity scores and paired (matched) peak counts
- FBMN molecular network with spectral pairing annotations of common-ion nodes

## How to apply

Preprocess the LC-MS/MS data with MZmine 2.53 (m/z tolerance 0.001 m/z or 15 ppm), exporting the feature quantification table (.csv, peak area columns) with 'Export/Submit to GNPS-FBMN: Filter rows: Only with MS2' and the paired fragmentation spectra list (.mgf). Import both files into the GNPS2 platform and run the Feature Based Molecular Network (FBMN) workflow so that common-ion nodes are matched against the curated in-house/GNPS2 spectral library, keeping the adduct consideration [M+H]+ at 15 ppm tolerance as used in the annotation layers. For each matched node, extract the library hit, the cosine similarity score, and the number of paired (matched) peaks, then compile a per-strain results table. This library search constitutes the first (spectral pairing) layer of the three-level annotation scheme; nodes without library matches are subsequently propagated with ChemWalker (adduct [M+H]+, 15 ppm) and then annotated in silico with SIRIUS 5.8.5.

## Related tools

- **MZmine 2.53** (Preprocesses full scan and MS/MS data separately (m/z tolerance 0.001 m/z or 15 ppm) and exports the .csv feature quantification table and .mgf spectra list fed to FBMN)
- **GNPS2 (FBMN workflow)** (Performs molecular networking and spectral pairing annotation of common-ion nodes against the curated in-house/GNPS2 spectral library)
- **GNPS library** (Alternative public library used for spectral pairing of SPL-derived spectra)
- **RegFilter** (Generates the scheduled precursor list (SPL) whose list-only ions are the spectra subsequently matched against the library) — https://github.com/computational-chemical-biology/regression_filter

## Evaluation signals

- The results table contains one cosine score and paired-peak count per strain whose values reproduce the reference pairs: 0.9565/25 peaks for BRA006, 0.3228/10 peaks for BRA010, and 0.6048/13 peaks for BRA177; any deviation requires expert review of the library match
- The exported .csv contains only rows with MS2 ('Export/Submit to GNPS-FBMN: Filter rows: Only with MS2') and pairs 1:1 with nodes in the .mgf
- Library matching uses the adduct consideration [M+H]+ at 15 ppm tolerance, consistent with the annotation layers of the workflow
- Matched nodes appear in the FBMN molecular network and their spectra derive from the SPL-acquired MS/MS data preprocessed with MZmine 2.53

## Limitations

- Low cosine scores with few paired peaks (e.g. 0.3228/10 for BRA010) indicate weak or questionable library matches that require expert review
- Spectral library matching only annotates the subset of common ions; ions selected only in the RegFilter scheduled precursor list and missed by DDA need additional annotation layers (ChemWalker, SIRIUS) to be recovered
- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, so spectra available for matching are incomplete

## Evidence

- [results] cosine scores and paired-peak counts of 0.9565/25 for BRA006, 0.3228/10 for BRA010, and 0.6048/13 for BRA177: [evidence span withheld — anchor status: not_found]
