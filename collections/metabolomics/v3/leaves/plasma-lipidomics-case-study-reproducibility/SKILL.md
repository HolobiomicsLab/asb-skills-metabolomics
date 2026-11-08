---
name: plasma-lipidomics-case-study-reproducibility
description: Use when validating MS2Query analogue-search predictions on new experimental
  lipidomics data (e.g. plasma-derived positive-mode MS2 spectra) to confirm matches
  consistently fall in the expected compound class.
when_to_use_negative:
- Query spectra are negative-mode or not derived from a lipid-rich biological matrix
  — compound-class consistency would not be expected to match the lipid benchmark
- The goal is de novo structure elucidation rather than library-based analogue/exact-match
  search against an annotated spectral library
- Spectra have not been peak-picked or clustered and contain many MS2 spectra per
  feature — README advises preprocessing with MZMine first
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2Query
  role: Runs the MS2Deepscore + random-forest analogue/exact-match search (version
    0.3.2) on the positive-mode plasma spectra
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Computes spectral similarity scores between all library spectra and each query
    spectrum for top-2000 candidate preselection
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Cleans metadata and peak lists (peaks >1000 Da and <0.1% of base peak removed,
    spectra with <3 peaks removed) before searching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:8cfad5c73e863af7e31eb3a25cfa3e5ec1584f6397bc24f100033a1ce9410671
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# plasma-lipidomics-case-study-reproducibility

## Summary

Reproduces the NIST 1950 Frozen Human Plasma lipid-profiling case study from the MS2Query paper, checking that 72 of 75 predicted matches from 139 positive-mode spectra belong to lipid compound classes.

## When to use

Use when validating MS2Query analogue-search predictions on new experimental lipidomics data (e.g. plasma-derived positive-mode MS2 spectra) to confirm matches consistently fall in the expected compound class.

## When NOT to use

- Query spectra are negative-mode or not derived from a lipid-rich biological matrix — compound-class consistency would not be expected to match the lipid benchmark
- The goal is de novo structure elucidation rather than library-based analogue/exact-match search against an annotated spectral library
- Spectra have not been peak-picked or clustered and contain many MS2 spectra per feature — README advises preprocessing with MZMine first

## Inputs

- 139 positive-mode MS2 query spectra from NIST 1950 Frozen Human Plasma SRM (Zenodo 10.5281/zenodo.6811540)
- Pre-trained MS2Query model files and preprocessed GNPS library spectra (Zenodo)
- Compound-class annotation mappings from LIPID MAPS, HMDB, and GNPS

## Outputs

- Per-spectrum results table ms2query_results_nist1950_positive.csv (75 predicted matches with compound classes)
- Case-study verification: 72 of 75 matches classified as lipids

## How to apply

Load the 139 positive-mode query spectra from the NIST 1950 plasma case study (Zenodo 10.5281/zenodo.6811540) and clean them with matchms: remove peaks above 1000 Da, peaks below 0.1% of the highest peak, and spectra with fewer than 3 peaks. Download the pre-trained MS2Query model and preprocessed GNPS library spectra from the Zenodo deposit. Run MS2Query 0.3.2 in positive mode so MS2Deepscore similarity scores are computed against all library spectra, the top 2000 candidates are preselected, and a random forest combining 5 features re-ranks them to identify exact matches and analogues. Attach compound-class annotations (LIPID MAPS / HMDB / GNPS mappings) to each predicted match and verify that 75 matches are predicted from 139 spectra, of which 72 are lipid-class matches.

## Related tools

- **MS2Query** (Runs the MS2Deepscore + random-forest analogue/exact-match search (version 0.3.2) on the positive-mode plasma spectra) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Computes spectral similarity scores between all library spectra and each query spectrum for top-2000 candidate preselection) — https://github.com/iomega/ms2query
- **matchms** (Cleans metadata and peak lists (peaks >1000 Da and <0.1% of base peak removed, spectra with <3 peaks removed) before searching)

## Examples

```
ms2query --spectra ./nist1950_positive_spectra --library ./ms2query_library --ionmode positive --results ./ms2query_results_nist1950_positive
```

## Evaluation signals

- Result table contains exactly 75 predicted matches from the 139 input spectra
- 72 of 75 matches are annotated as lipids using LIPID MAPS / HMDB / GNPS class mappings
- All query spectra had peaks above 1000 Da and below 0.1% of the highest peak removed, and spectra with fewer than 3 peaks were dropped
- MS2Deepscore preselection retains the top 2000 library spectra per query before random-forest re-ranking
- Output CSV is written as ms2query_results_nist1950_positive.csv with per-spectrum match annotations

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set, and manual validation of results is challenging
- MS2Query does not perform peak picking or clustering of similar MS2 spectra; many MS2 spectra per feature should first be reduced by clustering or feature selection
- Details of the benchmarking model for case studies are in Supplementary Note 1, which is not included in the extracted article text

## Evidence

- [discussion] Case study reproduces lipid-class consistency check: [evidence span withheld — anchor status: not_found]
- [discussion] Preprocessing thresholds: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] MS2Query two-stage workflow: "The top 2000 spectra with the highest MS2Deepscore are selected."
- [discussion] Data availability: "The raw data, intermediate files and raw results can be found on 10.5281/zenodo.6811540."
- [discussion] Compound-class annotation databases: "online databases LIPID MAPS48, HMDB49, and GNPS20"
- [readme] README: generalization caveat: [evidence span withheld — anchor status: external_doc]
