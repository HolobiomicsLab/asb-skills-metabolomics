---
name: lipid-compound-class-annotation-from-spectral-matches
description: Use when MS2Query (or a similar library-matching tool) has returned per-spectrum
  matches for complex-mixture metabolomics data and you must check whether annotations
  agree with the expected compound class (e.g., lipids in plasma).
when_to_use_negative:
- Query data are not from a mixture with a known expected compound-class composition
  — there is no reference class expectation to validate against
- 'Input spectra have not been preprocessed: MS2Query does no peak picking or clustering,
  and raw files with many MS2 spectra per feature should first be reduced (e.g. MZMine
  FBMN output)'
- The goal is exact structural elucidation rather than class-level consistency checking
  — MS2Query predicts analogues and exact matches, not de novo structures
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MS2Query
  role: Runs the MS2Deepscore preselection (top 2000) plus random-forest re-ranking
    to predict exact matches and analogues whose compound classes are then annotated
  repo: https://github.com/iomega/ms2query
- name: MS2Deepscore
  role: Calculates spectral similarity scores between all library spectra and each
    query spectrum for candidate preselection
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Cleans metadata and peak lists of query spectra (peaks >1000 Da and <0.1%
    relative intensity removed; spectra with <3 peaks removed)
  repo: https://github.com/iomega/ms2query
- name: LIPID MAPS
  role: Online database used to attach lipid compound-class annotations to predicted
    matches
- name: HMDB
  role: Online database used alongside LIPID MAPS and GNPS to attach compound-class
    annotations to predicted matches
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:8f73f2b5b9e21a6427f69ad9a1eaa13052cede669b8b33f95f679f8338dceacb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - HMDB
  - LIPID MAPS
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# lipid-compound-class-annotation-from-spectral-matches

## Summary

Attach compound-class annotations (lipid vs non-lipid) to MS2Query spectral matches using LIPID MAPS, HMDB, and GNPS mappings, then verify that predicted matches consistently belong to the expected compound class, as demonstrated on the NIST 1950 plasma case study where 72 of 75 matches were lipids.

## When to use

Use when MS2Query (or a similar library-matching tool) has returned per-spectrum matches for complex-mixture metabolomics data and you must check whether annotations agree with the expected compound class (e.g., lipids in plasma).

## When NOT to use

- Query data are not from a mixture with a known expected compound-class composition — there is no reference class expectation to validate against
- Input spectra have not been preprocessed: MS2Query does no peak picking or clustering, and raw files with many MS2 spectra per feature should first be reduced (e.g. MZMine FBMN output)
- The goal is exact structural elucidation rather than class-level consistency checking — MS2Query predicts analogues and exact matches, not de novo structures

## Inputs

- Positive-mode MS2 query spectra (mzML, MGF, msp, mzXML, json, USI, or pickled matchms object) from a complex biological mixture, e.g. NIST 1950 Frozen Human Plasma SRM
- Pre-trained MS2Query model files and preprocessed library spectra downloaded from Zenodo (10.5281/zenodo.6811540, positive mode record 6124552)
- Compound-class databases/mappings: LIPID MAPS, HMDB, GNPS

## Outputs

- Per-spectrum results CSV (e.g. ms2query_results_nist1950_positive.csv) listing predicted library matches with random-forest scores
- Compound-class annotations attached to each predicted match (lipid vs non-lipid classification)
- Case-study validation metrics (number of matches predicted, fraction belonging to expected compound class)

## How to apply

Run MS2Query (version 0.3.2) on cleaned positive-mode query spectra — for the NIST 1950 plasma case study, 139 spectra cleaned with matchms (peaks above 1000 Da removed, peaks below 0.1% of the highest peak removed, spectra with fewer than 3 peaks removed) against the preprocessed GNPS-based library from Zenodo deposit 10.5281/zenodo.6811540. MS2Query first scores all library spectra with MS2Deepscore, preselects the top 2000 candidates, then re-ranks them with a random forest combining 5 features; applying a minimum score threshold filters out unreliable matches. After the run, map each predicted match to compound-class information from LIPID MAPS, HMDB, and GNPS and write the per-spectrum results table (e.g., ms2query_results_nist1950_positive.csv). Judge correctness by comparing predicted-match composition against expected biology: the reported benchmark is 75 matches from 139 spectra, of which 72 were lipid-class matches, consistent with a plasma lipidome.

## Related tools

- **MS2Query** (Runs the MS2Deepscore preselection (top 2000) plus random-forest re-ranking to predict exact matches and analogues whose compound classes are then annotated) — https://github.com/iomega/ms2query
- **MS2Deepscore** (Calculates spectral similarity scores between all library spectra and each query spectrum for candidate preselection) — https://github.com/iomega/ms2query
- **matchms** (Cleans metadata and peak lists of query spectra (peaks >1000 Da and <0.1% relative intensity removed; spectra with <3 peaks removed)) — https://github.com/iomega/ms2query
- **LIPID MAPS** (Online database used to attach lipid compound-class annotations to predicted matches)
- **HMDB** (Online database used alongside LIPID MAPS and GNPS to attach compound-class annotations to predicted matches)

## Examples

```
ms2query --spectra ./nist1950_positive --library ./ms2query_library --ionmode positive
```

## Evaluation signals

- Match count consistency: 75 matches predicted from 139 positive-mode NIST 1950 plasma spectra
- Class-consistency rate: 72 of 75 predicted matches belong to lipid classes, matching the reported case-study metric
- All query spectra pass the matchms cleaning invariants: no peaks above 1000 Da, no peaks below 0.1% of the highest peak, no spectra with fewer than 3 peaks
- Random-forest match scores fall between 0 and 1, and matches below the minimum score threshold are filtered out as unreliable
- Results CSV schema includes per-spectrum matches in the output directory created alongside the query spectra

## Limitations

- It is hard to objectively judge whether MS2Query performs similarly on newly generated data compared to the benchmarking test set, and manual validation of results is challenging
- The correct compound class may be known while the exact identity is not — class-level consistency does not confirm exact structural matches
- MS2Query does no peak picking or clustering of similar MS2 spectra; files with many MS2 spectra per feature must be preprocessed first (e.g. with MZMine)

## Evidence

- [discussion] NIST 1950 plasma class consistency: [evidence span withheld — anchor status: not_found]
- [discussion] random forest ranking: "To optimally rank these 2000 spectra, MS2Query calculates 5 features which are combined by a random forest model."
- [discussion] peak cleaning thresholds: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] class databases: "online databases LIPID MAPS48, HMDB49, and GNPS20"
- [readme] readme filtering unreliable matches: [evidence span withheld — anchor status: external_doc]
