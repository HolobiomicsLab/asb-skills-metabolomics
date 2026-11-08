---
name: msms-cosine-similarity-library-matching
description: Use when untargeted LC–MS/MS feature tables carry acquired MS2 spectra
  for a subset of features and you want per-annotation posterior probabilities that
  exploit the fragmentation pattern, not just MS1 peak masses and adducts.
when_to_use_negative:
- When no MS2 data exist — in that case the article directs you to use MS1annotation()
  instead of MSMSannotation()
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: ipaPy2
  role: Provides the MSMSannotation() function that scores library candidates by cosine
    similarity of the fragmentation pattern and updates prior/posterior annotation
    probabilities
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Loads the MS1 features, MS2 spectra and library databases into dataframes
    required by MSMSannotation()
- name: numpy
  role: Supports the numeric handling of cosine similarity values and probability
    calculations
- name: mzMatch
  role: Upstream data-processing pipeline whose output can be integrated into the
    ipaPy2 annotation workflow
  repo: https://github.com/UoMMIB/mzmatch.R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:e85a3d03b94f0ef38f0c9ff65f23c5b51b6ea7fd038a0d245d451475b49ada01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - Python
  - ipaPy2
  - mzMatch
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# msms-cosine-similarity-library-matching

## Summary

Integrates tandem MS fragmentation spectra into the IPA Bayesian annotation pipeline, scoring library candidates against acquired MS2 spectra by cosine similarity so that MS1+MS2 posterior annotation probabilities supersede MS1-only ones.

## When to use

Use when untargeted LC–MS/MS feature tables carry acquired MS2 spectra for a subset of features and you want per-annotation posterior probabilities that exploit the fragmentation pattern, not just MS1 peak masses and adducts.

## When NOT to use

- When no MS2 data exist — in that case the article directs you to use MS1annotation() instead of MSMSannotation()

## Inputs

- Pandas dataframe of MS1 features (e.g., from ExampleDatasets/README/df_test_pos.csv), already passed through map_isotope_patterns() and MS1annotation()
- Pandas dataframe of acquired MS2 spectra (e.g., from ExampleDatasets/README/MS2data_example.csv)
- Reduced MS2 library database (e.g., DB/DBMS2_test_pos.csv, MoNa-derived QExactive spectra)
- All-adducts table produced by compute_all_adducts(adducts, DB, ionisation=1)
- Adducts file DB/adducts.csv

## Outputs

- Annotation dataframe/dictionary with prior and posterior probabilities per (feature id, compound id, adduct), now incorporating the fragmentation cosine similarity score
- Per-annotation fragmentation pattern (cosine similarity) scores for features with acquired MS2 spectra

## How to apply

After computing the all-adducts table with compute_all_adducts() on a reduced database (e.g., DB/DB_test_pos.csv with its MS2 counterpart DB/DBMS2_test_pos.csv), pass the MS2 dataframe loaded from 'ExampleDatasets/README/MS2data_example.csv' together with the MS1annotated dataframe into MSMSannotation(). The function matches each feature's acquired fragmentation spectra against the library entries using a cosine similarity of the peak patterns and integrates that score with the MS1 evidence; the CSunk=0.7 parameter sets the cosine threshold used to separate confident matches from unknown spectra. Run it with ppm=3 mass tolerance and ncores=1 in the README example, then join the resulting annotation dictionary with the MS1-only dictionary per (feature id, compound id, adduct) to quantify the prior and posterior probability shifts. Correct application means every feature with an acquired MS2 spectrum carries a non-empty fragmentation score, and posterior probabilities move toward library candidates that reproduce the observed fragment peaks.

## Related tools

- **ipaPy2** (Provides the MSMSannotation() function that scores library candidates by cosine similarity of the fragmentation pattern and updates prior/posterior annotation probabilities) — https://github.com/francescodc87/ipaPy2
- **pandas** (Loads the MS1 features, MS2 spectra and library databases into dataframes required by MSMSannotation())
- **numpy** (Supports the numeric handling of cosine similarity values and probability calculations)
- **mzMatch** (Upstream data-processing pipeline whose output can be integrated into the ipaPy2 annotation workflow) — https://github.com/UoMMIB/mzmatch.R

## Examples

```
annotations = ipaPy2.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1)
```

## Evaluation signals

- Every feature id that has acquired MS2 spectra must carry a non-empty fragmentation pattern (cosine similarity) score in the MSMSannotation output
- Joining MS1-only and MS1+MS2 annotation dictionaries on (compound id, adduct) yields a table with prior and posterior values from both runs; the deltas should show improved posterior probabilities for candidates whose fragments match the observed spectra
- The MSMSannotation call must use the same ppm=3 tolerance and the same all-adducts table as the MS1-only run so that probability differences are attributable to the fragmentation score, not a changed parameter set
- Entries in the MS2 library database must correspond to the 'MS2' ids present in the MS1 database file; an empty mapping signals a malformed library input
- The fragmentation scores should fall within the [0, 1] cosine similarity range, with CSunk=0.7 acting as the known unknowns confidence threshold

## Limitations

- Requires the MS2 library to be reduced to compounds with at least one QExactive fragmentation spectrum in MoNa; spectra from other instruments will not be matched
- Cosine similarity depends on the quality and completeness of the acquired MS2 spectra; features without spectra fall back to MS1-only probabilities
- Exotic adducts or in-source fragments not present in DB/adducts.csv must be manually added before annotation, or the fragmentation matching will omit them

## Evidence

- [abstract] Article abstract: "The revised method is now able to integrate tandem MS fragmentation data, which increases the accuracy of the identifications."
- [methods] Article methods — function description: "fragmentation data can be included in the annotation process by using the MSMSannotation() function"
- [methods] Article methods — MS1-only fallback: "If no fragmentation data is available, the MS1annotation() function should be used"
- [readme] README — example database note: "a reduced example database is also provided"
- [readme] README — MoNa library provenance: "contains all the MS2 spectra found in the [MoNa](https://mona.fiehnlab.ucdavis.edu) database acquired with a QExactive"
- [other] Source card task_006 workflow: [evidence span withheld — anchor status: not_found]
