---
name: msms-spectral-library-analogue-search
description: Use when you have cleaned positive- or negative-mode MS2 query spectra
  (mzML, MGF, msp, mzXML, json, USI, or pickled matchms) and need reliable library
  matches and analogues beyond modified cosine search.
when_to_use_negative:
- Input is raw LC-MS data that still needs peak picking or MS2 clustering — MS2Query
  does no peak picking, so preprocess with e.g. MZMine FBMN first
- The spectra mix multiple ionization modes in one run — MS2Query requires a single
  --ionmode (positive or negative) per run with the matching library
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2Query
  role: 'Performs the exact-match and analogue search: MS2Deepscore preselection of
    top 2000 candidates followed by random-forest re-ranking with 5 features'
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Computes spectral similarity scores between all library spectra and each query
    via pre-calculated embeddings for candidate preselection
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Cleans metadata and peaks, normalizes intensities, and filters spectra with
    fewer than 3 peaks before searching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:a3a4e2af0a2f29bbd06f28b633a3953363368abab0c9693cdde2cb21231cd332
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msms-spectral-library-analogue-search

## Summary

Run MS2Query to search MS2 query spectra against a preprocessed GNPS mass spectral library in one run, finding both exact matches and analogues by combining MS2Deepscore preselection with a random-forest re-ranking of 5 features.

## When to use

Use when you have cleaned positive- or negative-mode MS2 query spectra (mzML, MGF, msp, mzXML, json, USI, or pickled matchms) and need reliable library matches and analogues beyond modified cosine search.

## When NOT to use

- Input is raw LC-MS data that still needs peak picking or MS2 clustering — MS2Query does no peak picking, so preprocess with e.g. MZMine FBMN first
- The spectra mix multiple ionization modes in one run — MS2Query requires a single --ionmode (positive or negative) per run with the matching library

## Inputs

- MS2 query spectra files (mzML, MGF, msp, mzXML, json, USI, or pickled matchms object) in a single ionization mode
- Pre-trained MS2Query library folder containing library spectra (sqlite), models, and pre-calculated MS2Deepscore/Spec2Vec embeddings
- Optional matchms-cleaned query spectra (peaks >1000 Da removed, <0.1% base-peak intensity removed, <3-peak spectra removed)

## Outputs

- CSV results table per query spectrum with best library match/analogue and random-forest scores between 0 and 1 (e.g. ms2query_results_nist1950_positive.csv)
- Optionally, per-spectrum compound-class annotations mapped via LIPID MAPS / HMDB / GNPS for validation

## How to apply

First clean the spectra with matchms: remove peaks above 1000 Da, remove peaks with intensity below 0.1% of the base peak, trim spectra with more than 500 peaks to their highest-intensity peaks, and drop spectra with fewer than 3 peaks. Download the pre-trained MS2Query library and model files (e.g. from Zenodo record 6124552 for positive mode) and run MS2Query with the matching --ionmode. MS2Query calculates MS2Deepscore similarity between each query and all library spectra using pre-computed embeddings, preselects the top 2000 library spectra without precursor m/z preselection, then re-ranks those candidates with a random forest combining 5 features to score matches between 0 and 1; apply a score threshold to filter out unreliable matches. For compound-class validation (e.g. the NIST 1950 plasma case study), attach LIPID MAPS/HMDB/GNPS class annotations to predicted matches and check they belong to the expected class.

## Related tools

- **MS2Query** (Performs the exact-match and analogue search: MS2Deepscore preselection of top 2000 candidates followed by random-forest re-ranking with 5 features) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Computes spectral similarity scores between all library spectra and each query via pre-calculated embeddings for candidate preselection) — https://github.com/iomega/ms2query
- **matchms** (Cleans metadata and peaks, normalizes intensities, and filters spectra with fewer than 3 peaks before searching)

## Examples

```
ms2query --spectra ./nist1950_spectra --library ./ms2query_library --ionmode positive
```

## Evaluation signals

- Reproduce the NIST 1950 plasma case study: 75 matches predicted from 139 positive-mode spectra, of which 72 were lipid-class matches
- Random-forest match scores fall in the 0–1 range; unreliable matches are removed by the minimum score threshold
- Benchmark comparison shows MS2Query outperforms cosine score, modified cosine score, and MS2Deepscore alone on exact matches and analogues
- Dummy-data test run (dummy_spectra.mgf) reproduces the expected_results_dummy_data.csv from the repository in under half a minute after library download

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set, and manual validation of results is challenging
- MS2Query does not perform peak picking or clustering of similar MS2 spectra; many MS2 spectra per feature must be reduced beforehand (e.g. via MZMine FBMN)
- Pretrained libraries are GNPS-based; the GNPS public mass spectral libraries contain only about 2.5% of known natural products, limiting coverage

## Evidence

- [discussion] MS2Query first uses MS2Deepscore to calculate spectral similarity scores between all library spectra and a query spectrum.: "The workflow for running MS2Query first uses MS2Deepscore to calculate spectral similarity scores between all library spectra and a query spectrum."
- [discussion] Top 2000 preselection and random forest re-ranking: "The top 2000 spectra with the highest MS2Deepscore are selected."
- [discussion] Peak filtering thresholds: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [results] Case-study validation metric: [evidence span withheld — anchor status: not_found]
- [readme] README CLI usage: [evidence span withheld — anchor status: external_doc]
- [readme] Score range and thresholding: [evidence span withheld — anchor status: external_doc]
