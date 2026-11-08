---
name: fragmentation-spectrum-database-matching
description: Use when LC–MS/MS untargeted metabolomics data include acquired fragmentation
  spectra and candidate annotations from an MS1 database must be ranked by integrating
  fragment matching with isotope-pattern, mass-accuracy, and adduct evidence.
when_to_use_negative:
- When no fragmentation data have been acquired — use MS1annotation() instead, as
  MSMSannotation() is only appropriate when MS2 spectra are available
- When the input feature table has not yet been isotope-pattern-mapped or adducts
  have not been computed — run map_isotope_patterns() and compute_all_adducts() first
- When features have not been clustered — ipaPy2 provides clusterFeatures() as a prior
  step from raw intensity tables
edam_operation: http://edamontology.org/operation_0404
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ipaPy2
  role: Provides MSMSannotation(), which performs cosine-similarity fragment matching
    and posterior probability integration
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Loads the feature table, adducts table, MS2 spectra, and MS2 database CSVs
- name: numpy
  role: Used alongside pandas for dataframe handling in the annotation workflow
- name: PeakMLViewerPy
  role: Explores the annotation results produced by the ipaPy2 pipeline
  repo: https://github.com/UoMMIB/PeakMLViewerPy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:b7ae75e69bb808608a6c257a06dd3170a41b4939a49a92d210b275efdaeb8187
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - PeakMLViewerPy
  - Python
  - ipaPy2
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0404
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# fragmentation-spectrum-database-matching

## Summary

Scores candidate LC–MS/MS annotations by comparing measured MS2 spectra against database spectra using cosine similarity, then integrates the resulting fragmentation pattern score with mass accuracy, isotope pattern, RT range, and prior knowledge into posterior annotation probabilities via ipaPy2's MSMSannotation().

## When to use

Use when LC–MS/MS untargeted metabolomics data include acquired fragmentation spectra and candidate annotations from an MS1 database must be ranked by integrating fragment matching with isotope-pattern, mass-accuracy, and adduct evidence.

## When NOT to use

- When no fragmentation data have been acquired — use MS1annotation() instead, as MSMSannotation() is only appropriate when MS2 spectra are available
- When the input feature table has not yet been isotope-pattern-mapped or adducts have not been computed — run map_isotope_patterns() and compute_all_adducts() first
- When features have not been clustered — ipaPy2 provides clusterFeatures() as a prior step from raw intensity tables

## Inputs

- Isotope-pattern-mapped feature table (pandas DataFrame with measured intensities, e.g., ExampleDatasets/README/df_test_pos.csv)
- Adducts table for the database compounds (output of compute_all_adducts(), e.g., allAddsPos for positive mode)
- Measured MS2 spectra dataframe (e.g., ExampleDatasets/README/MS2data_example.csv)
- MS2 database with fragmentation spectra (e.g., DB/DBMS2_test_pos.csv; MoNa QExactive-derived in the paper)
- Adducts definition file (DB/adducts.csv, defining name, calc, Charge, Mass, Ion_mode, Formula_add, Formula_ded, Multi)

## Outputs

- Dictionary keyed by feature id mapping to per-feature annotation DataFrames containing a 'fragmentation pattern score' column (cosine similarity values) and updated 'post' posterior probabilities
- Pickle file of the annotation dictionary (e.g., annotations_msms.pkl)

## How to apply

Load the isotope-pattern-mapped feature table (e.g., df_test_pos.csv), the adducts table produced by compute_all_adducts(), the measured MS2 spectra, and a reduced MS2 database (e.g., DBMS2_test_pos.csv) with pandas. Call ipa.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1) so that, for each feature, the measured spectrum is compared against every matching database spectrum within a ppmCS=10 fragment matching tolerance (default evfilt=False, so all collision energies are used). The highest cosine similarity between measured and database spectra is taken as the 'fragmentation pattern score'; candidates with no database fragmentation spectrum receive the dummy CSunk=0.7 score so they remain comparable. The function then recomputes posterior probabilities by integrating the fragmentation score with mass accuracy, RT range, isotope pattern score, and prior knowledge, returning a dictionary of per-feature annotation dataframes that should be pickled (e.g., annotations_msms.pkl) for downstream review.

## Related tools

- **ipaPy2** (Provides MSMSannotation(), which performs cosine-similarity fragment matching and posterior probability integration) — https://github.com/francescodc87/ipaPy2
- **pandas** (Loads the feature table, adducts table, MS2 spectra, and MS2 database CSVs)
- **numpy** (Used alongside pandas for dataframe handling in the annotation workflow)
- **PeakMLViewerPy** (Explores the annotation results produced by the ipaPy2 pipeline) — https://github.com/UoMMIB/PeakMLViewerPy

## Examples

```
ipa.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1)
```

## Evaluation signals

- Every feature with acquired fragmentation data has a non-empty 'fragmentation pattern score' entry (verifiable, e.g., for feature id=1 as in the tutorial)
- 'post' probabilities change relative to the MS1-only annotation after fragmentation evidence is integrated
- Features with no matching database MS2 spectrum carry the dummy CSunk=0.7 score, not a computed cosine value
- The returned object is a dictionary of per-feature pandas DataFrames, pickleable to annotations_msms.pkl
- The script runs without error using the documented parameter values (ppm=3, CSunk=0.7, ncores=1)

## Limitations

- Candidates with no fragmentation spectrum in the database receive the fixed CSunk dummy score, so their fragmentation evidence is not truly measured
- Cosine similarity comparisons default to ppmCS=10 fragment matching tolerance and use all collision energies (evfilt=False), which may be unsuitable for instrument-specific filtering
- The method's accuracy depends on the completeness and instrument match of the MS2 database (e.g., MoNa QExactive spectra in the paper's example)

## Evidence

- [methods] fragmentation data can be included in the annotation process: "fragmentation data can be included in the annotation process by using the MSMSannotation() function"
- [methods] highest cosine similarity as fragmentation pattern score: [evidence span withheld — anchor status: not_found]
- [methods] MS2 spectra loaded from CSV in tutorial: "dfMS2=pd.read_csv('ExampleDatasets/README/MS2data_example.csv')"
- [readme] MS2 database file for reduced example: "a drastically reduced version of it has been included within this repository, and can be found [here](DB/DBMS2_test_pos.csv)"
- [abstract] ipaPy2 integrates MS2 data to increase accuracy: "The revised method is now able to integrate tandem MS fragmentation data, which increases the accuracy of the identifications."
