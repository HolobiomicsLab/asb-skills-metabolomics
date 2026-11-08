---
name: nmr-raw-data-availability-assessment
description: Use when you need to quantify how many publicly available studies in
  a metabolomics repository (e.g. Metabolomics Workbench) use NMR, involve Homo sapiens,
  or expose raw vendor acquisition files, broken down by sample type or instrument
  vendor.
when_to_use_negative:
- When the goal is to convert or process an individual study's raw NMR files for analysis
  — use CloMet's study-id-based retrieval instead ('by just inserting a study id our
  users can get a ready-to-use data set').
- When the user's data are private/local files rather than public repository content
  — CloMet handles those via its LOCAL tab, not via repository survey.
- 'When planning classification re-analysis: if class metadata is not available alongside
  the metabolomics data in the repository, availability of raw data alone does not
  make re-analysis feasible.'
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Metabolomics Workbench REST API
  role: Source of the study catalog; its study endpoints are queried and filtered
    to compute the survey counts
- name: CloMet
  role: Platform whose repository survey (Figure 3, right column) defines the target
    counts this skill reproduces; harmonizes NMR raw data file systems
  repo: github.com/rmallol/clomet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:622831bcf702a1fd630f51ad6c9678b086b31507dd437d1ac54897547fe63120
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Metabolomics Workbench REST API
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nmr-raw-data-availability-assessment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A survey procedure that quantifies raw-data availability in a public metabolomics repository by querying the Metabolomics Workbench REST API and stratifying studies by analysis type, organism, sample type, and instrument vendor, as reported in the CloMet paper's Figure 3 (right column).

## When to use

Use when you need to quantify how many publicly available studies in a metabolomics repository (e.g. Metabolomics Workbench) use NMR, involve Homo sapiens, or expose raw vendor acquisition files, broken down by sample type or instrument vendor.

## When NOT to use

- When the goal is to convert or process an individual study's raw NMR files for analysis — use CloMet's study-id-based retrieval instead ('by just inserting a study id our users can get a ready-to-use data set').
- When the user's data are private/local files rather than public repository content — CloMet handles those via its LOCAL tab, not via repository survey.
- When planning classification re-analysis: if class metadata is not available alongside the metabolomics data in the repository, availability of raw data alone does not make re-analysis feasible.

## Inputs

- Metabolomics Workbench REST API study catalog (JSON) from https://www.metabolomicsworkbench.org/rest
- Per-study metadata: analysis type, organism, sample type, and instrument/vendor fields

## Outputs

- workbench_survey_counts.json containing reproduced counts: total studies (1592), NMR studies (166), human NMR studies (61), raw-data studies (41), plasma (9), serum (7), Bruker (33), Agilent/Varian (8)
- Breakdown dimensions (organism, sample type, instrument vendor) annotated per count

## How to apply

Query the Metabolomics Workbench REST API (https://www.metabolomicsworkbench.org/rest) study endpoints to retrieve the full catalog of publicly available studies in JSON form, and count total studies (expected 1592 in the CloMet survey). Filter by analysis type 'NMR' to count NMR studies (166), then filter that subset by organism 'Homo sapiens' (61). Within the human NMR subset, identify studies providing raw vendor acquisition files (41) and stratify them by sample type — blood plasma (9) and blood serum (7) — and by instrument vendor, Bruker (33) vs Agilent/Varian (8). Write all reproduced counts and their breakdown dimensions to workbench_survey_counts.json; any mismatch against the paper's numbers requires re-checking the API query filters. The rationale is that raw data in these repositories lack standardization in file system format, so counting raw acquisition-file availability reveals how much of the corpus is actually reusable by analysis platforms.

## Related tools

- **Metabolomics Workbench REST API** (Source of the study catalog; its study endpoints are queried and filtered to compute the survey counts)
- **CloMet** (Platform whose repository survey (Figure 3, right column) defines the target counts this skill reproduces; harmonizes NMR raw data file systems) — github.com/rmallol/clomet

## Examples

```
curl -s https://www.metabolomicsworkbench.org/rest/study/study_id/ST000001/analysis; python scripts/count_studies.py --catalog studies.json --filters NMR,homo_sapiens,raw_vendor --out workbench_survey_counts.json
```

## Evaluation signals

- Total study count from the REST API equals 1592.
- NMR-filtered count equals 166, and the Homo sapiens subset of it equals 61.
- Raw-data subset of human NMR studies equals 41, stratifying exactly as 9 blood plasma and 7 blood serum.
- Instrument-vendor breakdown of selected experiments equals 33 Bruker and 8 Agilent/Varian.
- workbench_survey_counts.json parses as JSON and contains every breakdown dimension (analysis type, organism, sample type, vendor) alongside each count.

## Limitations

- Counts are a point-in-time snapshot from the 2023 CloMet survey; the live repository will grow, so exact equality (e.g. 1592) may fail on later re-runs and requires re-checking the API query filters rather than assuming the paper is wrong.
- The raw-data classification depends on whether vendor acquisition files (Bruker or Agilent/Varian) are actually deposited; raw data in the repositories lack standardization in file system format.
- Raw-data availability alone is insufficient for classification re-analysis when class information is not available along with the metabolomics data in public repositories.

## Evidence

- [other] Metabolomics Workbench REST API study endpoints to retrieve the full catalog of publicly available studies in JSON form: [evidence span withheld — anchor status: not_found]
