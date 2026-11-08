---
name: msms-intensity-coverage-filtering
description: Use when curating raw MS/MS spectral libraries (NIST'17, MS-DIAL, MSnLib
  .mgf/.msp entries) into training-ready data for in silico fragmentation models and
  uncurated spectra must be quality-controlled by fragment-to-peak matching criteria.
when_to_use_negative:
- When working with already-curated training data (e.g., a released library.csv from
  the FIORA Zenodo record) — the filters have already been applied
- When the goal is predicting spectra with a trained FIORA model (fiora-predict) rather
  than curating training libraries
- When spectra cannot be fragment-matched at all (no generated single-edge fragments),
  since the >=2 peak-match criterion cannot be evaluated
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Source repository whose data-preparation scripts implement fragment generation,
    50 ppm peak matching, and the intensity-coverage / precursor-intensity retention
    filters
  repo: https://github.com/BAMeScience/fiora
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:2ba1a69d28b1828fa058219dfaa5c88aeb18bf5f55bec9d2810cc9ecf2577b41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - FIORA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# MS/MS intensity-coverage filtering

## Summary

A quality-control filter step from the FIORA spectral-library curation pipeline that retains MS/MS library spectra only when generated single-edge fragments match at least two observed peaks, cover a minimum of 50% of total peak intensity, and the precursor does not dominate with more than 90% of total intensity.

## When to use

Use when curating raw MS/MS spectral libraries (NIST'17, MS-DIAL, MSnLib .mgf/.msp entries) into training-ready data for in silico fragmentation models and uncurated spectra must be quality-controlled by fragment-to-peak matching criteria.

## When NOT to use

- When working with already-curated training data (e.g., a released library.csv from the FIORA Zenodo record) — the filters have already been applied
- When the goal is predicting spectra with a trained FIORA model (fiora-predict) rather than curating training libraries
- When spectra cannot be fragment-matched at all (no generated single-edge fragments), since the >=2 peak-match criterion cannot be evaluated

## Inputs

- Raw MS/MS spectral library entries from NIST'17, MS-DIAL (.msp), and MSnLib (*_ms2.mgf files from Zenodo)
- Per-compound generated single-edge fragments (with up to four hydrogen losses per bond cleavage)
- Peak-matching results at 50 ppm m/z tolerance (10 ppm for MSnLib) with intensity-coverage statistics

## Outputs

- Curated, quality-controlled spectral table (library.csv) containing only entries passing >=2 peak-match, >=50% intensity-coverage, and <=90% precursor-intensity criteria
- Set of excluded spectra (missing collision energies, unclear formats, failed coverage filters)

## How to apply

First parse raw library entries (converting NCE collision energies to eV, filtering for [M+H]+ and [M-H]- ionization types, plus [M]+/[M]- for MSnLib) and drop compounds with SMILES/MOL/InChIKey inconsistencies or exceeding 1000 Da / 100 eV hard limits. For each retained compound, generate single-edge fragments by evaluating each bond cleavage independently (allowing up to four hydrogen losses distributing charge between charged fragment and neutral loss), then match fragments to observed peaks at a 50 ppm m/z tolerance (10 ppm for MSnLib). Apply the hard retention filters: at least two peak matches, minimum peak intensity coverage of 50%, and maximum precursor intensity of 90%. Optionally apply the soft filters: 50% of total peaks or at least five peaks matched by fragments, or an exceptionally high intensity coverage of 0.8. Retained spectra are written to library.csv for FIORA training.

## Related tools

- **FIORA** (Source repository whose data-preparation scripts implement fragment generation, 50 ppm peak matching, and the intensity-coverage / precursor-intensity retention filters) — https://github.com/BAMeScience/fiora

## Examples

```
python resources/data/msnlib/download_msnlib.py && python resources/data/msnlib/preprocess_msnlib.py  # raw *_ms2.mgf -> curated library.csv with coverage filters applied
```

## Evaluation signals

- library.csv exists, is loadable as a table, and contains only entries within the 1000 Da weight and 100 eV collision-energy hard limits
- Every retained spectrum satisfies >=2 fragment-to-peak matches, >=50% peak intensity coverage, and <=90% precursor intensity
- Only [M+H]+ and [M-H]- (plus [M]+/[M]- for MSnLib) ionization types appear in the retained set
- No retained entries have missing or unclearly formatted collision energies (all converted to eV)
- Downstream FIORA training on the filtered library converges without entries being rejected by input validation

## Limitations

- The filters depend on generated single-edge fragments only; multi-step fragmentation is not covered, limiting the fragment vocabulary available for matching
- MSnLib uses a tighter 10 ppm matching tolerance than the 50 ppm tolerance used for NIST'17 and MS-DIAL, so thresholds are not uniform across sources
- Only entries with parseable collision energies (converted from NCE to eV) can be filtered; spectra with missing or unclear CE formats are excluded upstream

## Evidence

- [discussion] Hard retention filters: "We enforced at least two peak matches, a minimum peak intensity coverage of 50%, and a maximum precursor intensity of 90%."
- [discussion] Soft filter alternatives: "soft filters required that each spectrum have either 50% of the total peaks or at least five peaks matched by fragments, or an exceptionally high intensity coverage of 0.8."
- [discussion] Upstream hard limits: "These requirements included a maximum weight of 1000 Da and maximum collision energy of 100 eV."
- [discussion] Collision energy completeness requirement: "MS/MS spectra with missing collision energies or unclear formats were excluded."
- [readme] Raw MSnLib acquisition: "The downloader defaults to MSnLib v7 on Zenodo (`https://zenodo.org/records/16984129`) and accepts both direct file URLs and Zenodo record URLs."
