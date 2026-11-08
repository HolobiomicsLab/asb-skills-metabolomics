---
name: cosine-similarity-scoring
description: Use when MZmine-exported feature tables (.csv with MS2-filtered rows)
  and paired fragmentation spectra (.mgf) need library annotation, and cosine scores
  plus matched-peak counts must be extracted per strain or sample node.
when_to_use_negative:
- Input spectra have not been pre-processed with MZmine (no .csv feature table and
  .mgf fragmentation list available), since the FBMN workflow requires both exports
- The goal is de novo or in silico annotation rather than library matching — use a
  tool such as SIRIUS for the in silico annotation layer instead
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS2
  role: Platform hosting the Feature Based Molecular Network (FBMN) workflow that
    performs spectral pairing annotation against the curated in-house/GNPS2 spectral
    library
- name: MZmine 2.53
  role: Pre-processes converted .mzXML files (m/z tolerance 0.001 m/z or 15 ppm) and
    exports the feature table (.csv, rows with MS2) and fragmentation spectra (.mgf)
    consumed by FBMN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:33b646f2e16e7c32820a3ca3b6f5cca8ee071a29fbcc137a7df641f947222919
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
  - MZmine
  - MZmine 2.53
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cosine-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates fragment spectra by spectral pairing against a curated in-house/GNPS2 spectral library via the GNPS2 Feature Based Molecular Network (FBMN) workflow, reporting cosine similarity scores and paired-peak counts per feature node.

## When to use

Use when MZmine-exported feature tables (.csv with MS2-filtered rows) and paired fragmentation spectra (.mgf) need library annotation, and cosine scores plus matched-peak counts must be extracted per strain or sample node.

## When NOT to use

- Input spectra have not been pre-processed with MZmine (no .csv feature table and .mgf fragmentation list available), since the FBMN workflow requires both exports
- The goal is de novo or in silico annotation rather than library matching — use a tool such as SIRIUS for the in silico annotation layer instead

## Inputs

- MZmine-exported feature quantification table (.csv, peak area columns, rows filtered to features with MS2)
- MZmine-exported paired fragmentation spectra list (.mgf)
- Curated in-house / GNPS2 spectral library

## Outputs

- Per-node library match table with cosine similarity scores and paired-peak counts (e.g. 0.9565/25 peaks for BRA006, 0.3228/10 for BRA010, 0.6048/13 for BRA177)
- FBMN molecular network with spectral pairing annotations

## How to apply

Pre-process the LC-MS/MS data with MZmine 2.53 using 0.001 m/z or 15 ppm tolerances, then export the feature quantification table filtered to rows with MS2 ('Export/Submit to GNPS-FBMN: Filter rows: Only with MS2') and the paired fragmentation spectra list as .mgf. Import both files into the GNPS2 FBMN workflow so each common-ion node is paired against the curated in-house/GNPS2 spectral library, keeping the adduct consideration [M+H]+ at 15 ppm tolerance as used in the annotation layers. For each matched node, extract the library hit, the cosine similarity score, and the number of paired (matched) peaks. Compile per-sample results and compare against reference pairs; note that cosine scores range from strong matches (e.g. 0.9565 with 25 peaks) to weak ones (e.g. 0.3228 with 10 peaks), so low scores warrant expert review rather than automatic rejection.

## Related tools

- **GNPS2** (Platform hosting the Feature Based Molecular Network (FBMN) workflow that performs spectral pairing annotation against the curated in-house/GNPS2 spectral library)
- **MZmine 2.53** (Pre-processes converted .mzXML files (m/z tolerance 0.001 m/z or 15 ppm) and exports the feature table (.csv, rows with MS2) and fragmentation spectra (.mgf) consumed by FBMN)

## Evaluation signals

- Results table contains a cosine score and paired-peak count row for each of the three strains matching the reference pairs: 0.9565/25 for BRA006, 0.3228/10 for BRA010, 0.6048/13 for BRA177
- Adduct considered is [M+H]+ with 15 ppm tolerance, matching the annotation layers of the workflow
- Feature table input contains only rows with MS2, per the 'Export/Submit to GNPS-FBMN: Filter rows: Only with MS2' setting
- Cosine scores fall in [0, 1] and paired-peak counts are integers; any deviation from the reported reference pairs requires expert review of the library match

## Limitations

- Low cosine scores (e.g. 0.3228 with 10 paired peaks for BRA010) indicate weak library matches that require expert review; annotation confidence varies per node
- Annotation depends on the coverage of the curated library — RegFilter's workflow 'offers the possibility of creating local spectral libraries curated according to the user's interests', so uncurated compounds may lack matches

## Evidence

- [results] Searching SPL-derived spectra against the curated in-house library yielded cosine scores and paired-peak counts of 0.9565/25 for BRA006, 0.3228/10 for BRA010, and 0.6048/13 for BRA177.: [evidence span withheld — anchor status: not_found]
- [methods] the molecular network and spectral pairing annotation was performed by the Feature Based Molecular Network (FBMN) workflow (Nothias et al., 2020): "the molecular network and spectral pairing annotation was performed by the Feature Based Molecular Network (FBMN) workflow (Nothias et al., 2020)"
- [results] we performed their spectral pairing using the GNPS2 library (Wang et al., 2016), as it comes from one of the largest public libraries of spectral data: "we performed their spectral pairing using the GNPS2 library (Wang et al., 2016), as it comes from one of the largest public libraries of spectral data"
- [methods] Export/Submit to GNPS-FBMN: Filter rows: Only with MS2: "Export/Submit to GNPS-FBMN: Filter rows: Only with MS2"
- [abstract] the tool offers the possibility of creating local spectral libraries curated according to the user’s interests.: "the tool offers the possibility of creating local spectral libraries curated according to the user’s interests."
