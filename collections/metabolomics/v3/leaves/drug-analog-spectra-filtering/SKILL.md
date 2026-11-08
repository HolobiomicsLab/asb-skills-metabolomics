---
name: drug-analog-spectra-filtering
description: Use when converting raw drug-analog matches (from GNPS Drug Library spectra
  vs. untargeted metabolomics data) into a reliable propagated analog library, i.e.,
  when analog spectra must be filtered to keep only chemically explainable, non-duplicate,
  exposure-informative MS/MS hits.
when_to_use_negative:
- The spectrum already receives a full GNPS library match — it is a direct spectral
  hit, not a propagated analog, and belongs in direct screening rather than this analog-propagation
  step.
- The parent drug has an endogenous or dietary-source exposure annotation — these
  analogs are deliberately excluded by the first filter.
- The goal is immediate clinical decision-making about patient medication exposure
  — the GNPS Drug Library is explicitly not intended for that purpose.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Provides the Drug Library MS/MS reference spectra and library-match search
    that generate the raw analog matches to be filtered
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: microbe_masst (domainMASSTs batch search)
  role: Batch-searches analog spectra against indexed public metabolomics datasets
    (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN) via the Fast Search
    API to detect over-detected analogs in the dataset-testing filter
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:0519dec9badca5edf5eff954edafa61f749078e8d502c03f1eb939982923c42c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - microbe_masst (domainMASSTs batch search)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# drug-analog-spectra-filtering

## Summary

Applies four curation filters (endogenous/food-source removal, mass-offset filtering, library-match filtering, dataset testing) to raw drug-analog MS/MS matches from the GNPS Drug Library, yielding a curated propagated analog library for drug exposure screening in untargeted metabolomics.

## When to use

Use when converting raw drug-analog matches (from GNPS Drug Library spectra vs. untargeted metabolomics data) into a reliable propagated analog library, i.e., when analog spectra must be filtered to keep only chemically explainable, non-duplicate, exposure-informative MS/MS hits.

## When NOT to use

- The spectrum already receives a full GNPS library match — it is a direct spectral hit, not a propagated analog, and belongs in direct screening rather than this analog-propagation step.
- The parent drug has an endogenous or dietary-source exposure annotation — these analogs are deliberately excluded by the first filter.
- The goal is immediate clinical decision-making about patient medication exposure — the GNPS Drug Library is explicitly not intended for that purpose.

## Inputs

- Raw drug-analog match table from searching GNPS Drug Library MS/MS spectra against public metabolomics data
- Curated drug metadata (exposure sources, pharmacologic classes) for flagging endogenous or dietary-source drugs
- List of 156 curated, chemically explained mass offsets
- GNPS library match results (to identify spectra with full library hits)
- 20 public test metabolomics datasets (via MASST-style fast search) for the over-detection filter

## Outputs

- analog_library_filtered.csv — 12,455 clustered, non-duplicated analog MS/MS spectra covering 1277 drugs
- Per-filter exclusion records for endogenous/food-source, mass-offset, library-match, and dataset-testing removals

## How to apply

Load the raw drug-analog match table produced from GNPS Drug Library spectra together with curated metadata on exposure sources and pharmacologic classes. First exclude analogs whose parent drug carries an endogenous or dietary-source exposure annotation, then exclude analogs whose mass offset is not among the 156 curated, chemically explained mass offsets. Next remove analog spectra that already receive a full GNPS library match, since those are direct spectral hits rather than propagated analogs, and remove analogs that are over-detected (too ubiquitous) when searched against 20 public test metabolomics datasets. Finally, propagate the surviving analog spectra into a clustered analog library and write analog_library_filtered.csv. Validate against the paper's reported result of 12,455 clustered non-duplicated MS/MS spectra covering 1277 drugs.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Provides the Drug Library MS/MS reference spectra and library-match search that generate the raw analog matches to be filtered) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **microbe_masst (domainMASSTs batch search)** (Batch-searches analog spectra against indexed public metabolomics datasets (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN) via the Fast Search API to detect over-detected analogs in the dataset-testing filter) — https://github.com/robinschmid/microbe_masst

## Examples

```
python jobs.py  # add ("input.mgf", "output/prefix") entries to the files list; run with skip_existing=True repeatedly until no new output
```

## Evaluation signals

- Final output contains exactly 12,455 clustered non-duplicated analog MS/MS spectra, matching the paper's reported result
- All surviving analog spectra span 1277 distinct parent drugs, confirming coverage of the propagated library
- No remaining analog has a mass offset outside the 156 curated, chemically explained offsets
- No surviving analog spectrum duplicates a full GNPS library match; all direct hits were removed before propagation
- Over-detection check: surviving analogs are not ubiquitously matched across the 20 public test metabolomics datasets

## Limitations

- Over-detection is dataset-dependent: analogs too ubiquitous across public metabolomics data are removed, so coverage varies with how many and which datasets are indexed at search time.
- Existing MS/MS reference libraries mostly capture only parent drugs because drug metabolite reference standards are hard to source, so analog propagation relies on mass-offset reasoning rather than authenticated metabolite spectra.
- The resulting library is not intended for immediate clinical decision-making; it supports data-driven exploration of drug exposures only.

## Evidence

- [results] After applying four filters (endogenous/food-source removal, mass-offset filtering, library-match filtering, and dataset testing), the propagated analog library contains analogs of 1277 drugs: [evidence span withheld — anchor status: not_found]
- [methods] Exclude analogs whose mass offset is not among the 156 curated, chemically explained mass offsets.: [evidence span withheld — anchor status: not_found]
- [methods] Exclude analogs whose spectra already receive a full library match in the GNPS library, since those are direct spectral hits rather than propagated analogs.: [evidence span withheld — anchor status: not_found]
- [readme] A _datasets.tsv file will be generated. This contains the number of unique samples found to be matching your searched spectrum in each currently indexed dataset.: [evidence span withheld — anchor status: external_doc]
- [intro] existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards25, leaving a gap in the current drug: "existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards25, leaving a gap in the current drug"
- [abstract] While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable.: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
