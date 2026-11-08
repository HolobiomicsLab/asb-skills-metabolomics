---
name: exposure-source-metadata-filtering
description: Use when curating analog (non-library) MS/MS matches from untargeted
  metabolomics data and you must drop analogs whose parent drug can plausibly arise
  endogenously or from the food supply, before propagating spectra into a drug-exposure
  screening library.
when_to_use_negative:
- When the goal is direct spectral identification of parent drugs already present
  in the GNPS library — those are handled by library matching, not analog curation
  filters
- When screening for endogenous metabolites or food-derived compounds rather than
  exogenous drug exposure — this filter would incorrectly remove the signals of interest
- When the input analog matches already passed full curation (e.g., analog_library_filtered.csv)
  — re-filtering would be redundant
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Source of the Drug Library MS/MS reference spectra and of the standardized,
    machine-readable exposure-source and pharmacologic metadata used to flag endogenous
    or dietary-source parent drugs
  repo: ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:364a24f6bc75d49e8c78a30295a12edab86e20c335a13644e761062ea7300ff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# exposure-source-metadata-filtering

## Summary

A curation filter from the GNPS Drug Library pipeline that removes propagated drug-analog spectra whose parent drug carries an endogenous or dietary-source exposure annotation, so that downstream drug-exposure screening in untargeted metabolomics reflects true exogenous exposures rather than host- or food-derived chemistry.

## When to use

Use when curating analog (non-library) MS/MS matches from untargeted metabolomics data and you must drop analogs whose parent drug can plausibly arise endogenously or from the food supply, before propagating spectra into a drug-exposure screening library.

## When NOT to use

- When the goal is direct spectral identification of parent drugs already present in the GNPS library — those are handled by library matching, not analog curation filters
- When screening for endogenous metabolites or food-derived compounds rather than exogenous drug exposure — this filter would incorrectly remove the signals of interest
- When the input analog matches already passed full curation (e.g., analog_library_filtered.csv) — re-filtering would be redundant

## Inputs

- Raw drug-analog match table (experimental MS/MS spectra matched to GNPS Drug Library analogs)
- Curated drug metadata with standardized exposure-source annotations (endogenous vs. dietary vs. other) and pharmacologic classes
- GNPS Drug Library MS/MS reference spectra for drugs and their metabolites/analogs

## Outputs

- Filtered analog table restricted to analogs of drugs with non-endogenous, non-dietary exposure sources (contributing to analog_library_filtered.csv with 12,455 clustered spectra over 1277 drugs)

## How to apply

Load the raw drug-analog match table produced by searching experimental spectra against the GNPS Drug Library, together with curated metadata mapping each parent drug to its exposure sources and pharmacologic class. Flag analogs whose parent drug has an endogenous or dietary-source exposure annotation and exclude them, since such analogs cannot serve as evidence of external drug exposure. Apply this filter as the first of the four propagated-analog curation steps, before mass-offset filtering (against the 156 curated, chemically explained mass offsets), library-match filtering (removing analogs already getting a full GNPS library match), and dataset-testing (removing analogs over-detected across 20 public test metabolomics datasets). Propagate only analogs surviving all four filters into the final clustered library; the validated end state is 12,455 clustered non-duplicated MS/MS spectra covering analogs of 1277 drugs, written to analog_library_filtered.csv.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Source of the Drug Library MS/MS reference spectra and of the standardized, machine-readable exposure-source and pharmacologic metadata used to flag endogenous or dietary-source parent drugs) — ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Evaluation signals

- Output row count equals the published result: 12,455 clustered non-duplicated MS/MS spectra
- Number of unique parent drugs in the filtered analog library equals 1277
- No analog remaining in the output has a parent drug annotated as endogenous or dietary-source
- Remaining analogs' mass offsets are all among the 156 curated, chemically explained offsets, and none of their spectra receive a full GNPS library match
- Filtered table is written to analog_library_filtered.csv and every surviving analog passed all four curation filters (source, mass-offset, library-match, dataset-testing)

## Limitations

- The filter depends entirely on the accuracy of curated exposure-source annotations; drugs with missing or mislabeled source metadata may be wrongly kept or removed
- The GNPS Drug Library is not intended for immediate clinical decision-making; filtered analog matches are hypotheses of exposure, not diagnostic calls
- Analogs of drugs metabolized from food/endogenous precursors cannot be distinguished purely by metadata and may be lost or retained incorrectly

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] GNPS Drug Library contains a collection of MS/MS reference spectra for drugs and their metabolites/analogs along with standardized, machine-readable pharmacologic metadata: "GNPS Drug Library contains a collection of MS/MS reference spectra for drugs and their metabolites/analogs along with standardized, machine-readable pharmacologic metadata"
- [methods] Exclude analogs whose parent drug has an endogenous or dietary-source exposure annotation.: [evidence span withheld — anchor status: not_found]
- [results] After applying four filters (endogenous/food-source removal, mass-offset filtering, library-match filtering, and dataset testing), the propagated analog library contains analogs of 1277 drugs represented by 12,455 clustered non-duplicated MS/MS spectra.: [evidence span withheld — anchor status: not_found]
- [abstract] While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable.: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
