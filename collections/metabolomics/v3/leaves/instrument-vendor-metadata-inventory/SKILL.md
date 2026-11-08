---
name: instrument-vendor-metadata-inventory
description: Use when reproducing repository-level survey counts (e.g., Figure 3 of
  the CloMet paper) or auditing which instrument vendors appear in raw data of a public
  metabolomics repository's study catalog.
when_to_use_negative:
- When the goal is converting or standardizing raw NMR data files for analysis platforms
  like MetaboAnalyst or Workflows4Metabolomics — that is CloMet's data-conversion
  purpose, not metadata tallying.
- When per-spectrum signal processing is needed — the article states data processing
  algorithms are out of scope for the surveyed tooling.
- When the repository's study metadata lacks the fields needed for the filter chain
  (e.g., species or instrument vendor), since subsets cannot be restricted reproducibly.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboLights
  role: Source repository queried via its REST API to retrieve the study catalog and
    per-study technique, species, sample-type, and spectrometer-vendor metadata used
    in the tally chain.
- name: CloMet
  role: Open-source platform from the same article that harmonizes MetaboLights/Metabolomics
    Workbench file systems with a focus on NMR-based metabolomics; contextual companion
    for downstream reuse of the surveyed studies.
  repo: github.com/rmallol/clomet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:7aa409ad96ee30593392f907238cbd7cff66bb338157a76e359853418804449f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - MetaboLights
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# instrument-vendor-metadata-inventory

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Builds a quantitative inventory of instrument-vendor metadata (e.g., Bruker spectrometers) across a public metabolomics repository's study catalog, by filtering the MetaboLights study list through successive metadata restrictions (technique, species, sample type, raw-data presence) and tallying each stage.

## When to use

Use when reproducing repository-level survey counts (e.g., Figure 3 of the CloMet paper) or auditing which instrument vendors appear in raw data of a public metabolomics repository's study catalog.

## When NOT to use

- When the goal is converting or standardizing raw NMR data files for analysis platforms like MetaboAnalyst or Workflows4Metabolomics — that is CloMet's data-conversion purpose, not metadata tallying.
- When per-spectrum signal processing is needed — the article states data processing algorithms are out of scope for the surveyed tooling.
- When the repository's study metadata lacks the fields needed for the filter chain (e.g., species or instrument vendor), since subsets cannot be restricted reproducibly.

## Inputs

- MetaboLights REST API study catalog (full list of studies with technique, species, sample-type, and instrument metadata)
- MetaboLights per-study metadata records (species, sample type blood plasma/serum, raw data files, spectrometer vendor)

## Outputs

- survey_counts.csv with the four tallies (total studies, NMR studies, human NMR studies, human plasma/serum studies with raw Bruker data)
- Per-stage subset counts (887, 135, 56, 24) suitable for comparison against the article's Figure 3 left column

## How to apply

Query the MetaboLights REST API for the full study catalog and tally the total number of studies (expected: 887). Filter the catalog by measurement technique to isolate NMR-based studies and count them (expected: 135). Restrict the NMR subset to human studies using species metadata and record that count (expected: 56). Further restrict the human NMR subset to studies whose sample type is blood plasma or serum that include raw instrument data acquired on Bruker spectrometers, and record the final count (expected: 24). Write the four tallies to survey_counts.csv and compare each value against the numbers reported in the article's Figure 3 (left column), so every filtering decision is auditable against the published denominator chain (887 -> 135 -> 56 -> 24).

## Related tools

- **MetaboLights** (Source repository queried via its REST API to retrieve the study catalog and per-study technique, species, sample-type, and spectrometer-vendor metadata used in the tally chain.)
- **CloMet** (Open-source platform from the same article that harmonizes MetaboLights/Metabolomics Workbench file systems with a focus on NMR-based metabolomics; contextual companion for downstream reuse of the surveyed studies.) — github.com/rmallol/clomet

## Evaluation signals

- Total catalog count equals 887 studies.
- NMR-filtered subset count equals 135 studies.
- Human-restricted NMR subset count equals 56 studies.
- Final plasma/serum raw-data subset count equals 24 studies, and every study in that subset was acquired on a Bruker spectrometer (100% single-vendor invariant).
- survey_counts.csv contains exactly the four tallies, and each matches the corresponding value in the article's Figure 3 left column.

## Limitations

- Counts depend on depositor-supplied metadata; if species, class, or instrument information is not deposited with the study, the corresponding filter stage cannot be evaluated (the article notes that missing class metadata makes re-analysis infeasible).
- The tallies are a snapshot of the MetaboLights catalog at query time and may drift as studies are added, so exact equality with the published 887/135/56/24 values can degrade over time.
- The 24/24 Bruker result applies only to the human blood plasma/serum raw-data subset of NMR studies; it does not generalize to the full catalog or to other sample types/techniques.

## Evidence

- [other] Survey finding grounding the 24/24 Bruker invariant and the filter chain: [evidence span withheld — anchor status: not_found]
