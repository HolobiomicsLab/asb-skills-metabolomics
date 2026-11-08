---
name: species-and-sample-type-metadata-extraction
description: Use when the analysis goal is to stratify a public metabolomics repository
  catalog by measurement technique (e.g. NMR), species (e.g. human), and sample type
  (e.g. blood plasma/serum) to answer 'how many studies match X?' questions.
when_to_use_negative:
- You need to re-analyze raw spectral data for classification tasks — the paper notes
  this is infeasible when class metadata is not deposited alongside the data in public
  repositories.
- Your target studies lack per-sample species or sample-type annotations, so no reliable
  stratification can be performed.
- You need data analysis-ready spectral files rather than catalog-level counts — that
  is CloMet's conversion role, not metadata extraction.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboLights
  role: Source repository whose REST API provides the study catalog and per-study
    species/sample-type metadata being filtered
- name: CloMet
  role: Open-source platform that harmonizes repository metadata and file systems;
    the survey motivated its development for standardizing NMR-based MetaboLights
    data
  repo: https://github.com/rmallol/clomet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:3e55b9dd5ebfa833dff1baae0decd145e7e7a114b1b14837de6cfb56feda666a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - MetaboLights
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# species-and-sample-type-metadata-extraction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Extracts and filters repository-level study metadata (measurement technique, organism/species, sample type, instrument vendor) from a metabolomics repository catalog such as MetaboLights to produce stratified study counts, as demonstrated in the CloMet paper's survey of NMR-based MetaboLights studies.

## When to use

Use when the analysis goal is to stratify a public metabolomics repository catalog by measurement technique (e.g. NMR), species (e.g. human), and sample type (e.g. blood plasma/serum) to answer 'how many studies match X?' questions.

## When NOT to use

- You need to re-analyze raw spectral data for classification tasks — the paper notes this is infeasible when class metadata is not deposited alongside the data in public repositories.
- Your target studies lack per-sample species or sample-type annotations, so no reliable stratification can be performed.
- You need data analysis-ready spectral files rather than catalog-level counts — that is CloMet's conversion role, not metadata extraction.

## Inputs

- MetaboLights study catalog retrieved via the MetaboLights REST API (full list of studies)
- Per-study metadata records: measurement technique, species, sample type, raw instrument file/vendor information

## Outputs

- survey_counts.csv containing the four stratified tallies (total studies, NMR studies, human NMR studies, human NMR blood plasma/serum studies with raw Bruker data)
- Intermediate subset counts at each filter step (887 total, 135 NMR, 56 human, 24 plasma/serum Bruker)

## How to apply

Query the MetaboLights REST API for the full study catalog and tally the total number of studies. Filter the catalog by measurement technique — in this survey, NMR spectroscopy studies — and record the count. Within the NMR subset, use species metadata to restrict to human studies, then further restrict to studies whose sample type is blood plasma or serum that include raw instrument data acquired on Bruker spectrometers. Rationale: repository metadata (species, sample type, instrument) is the only tractable way to stratify public studies at scale, since re-analyzing raw data for classification is infeasible when class/sample metadata is absent — 'if information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible.' Record each intermediate tally so every filter step can be verified against published figures rather than only the final count.

## Related tools

- **MetaboLights** (Source repository whose REST API provides the study catalog and per-study species/sample-type metadata being filtered)
- **CloMet** (Open-source platform that harmonizes repository metadata and file systems; the survey motivated its development for standardizing NMR-based MetaboLights data) — https://github.com/rmallol/clomet

## Evaluation signals

- Total catalog tally matches the reported 887 MetaboLights studies at query time.
- NMR-technique filter yields 135 studies; species filter on that subset yields 56 human studies.
- Plasma/serum + raw Bruker filter yields 24 studies, matching the published figure.
- Invariant: all 24 final-subset studies report raw data acquired on Bruker spectrometers.
- Each intermediate count is recorded in survey_counts.csv and individually comparable to the article's reported numbers, not just the final tally.

## Limitations

- Counts are time-sensitive: the catalog size and stratification depend on the state of the MetaboLights repository when the API is queried.
- Stratification depends entirely on deposited metadata quality; missing class/sample-type annotations make deeper re-analysis infeasible.
- CloMet itself does not implement data processing algorithms — 'this is out of our scope' — so this skill yields catalog counts, not converted/processed spectra.

## Evidence

- [discussion] if information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible: [evidence span withheld — anchor status: not_found]
