---
name: collision-energy-spectrum-handling
description: Use when a feature table already has isotope-mapped entries and measured
  MS2 spectra (e.g. MS2data_example.csv) plus an MS2 database (DBMS2_test_pos.csv)
  are available, and you want to score candidate annotations by fragmentation pattern
  similarity.
when_to_use_negative:
- When no fragmentation data has been acquired for the features — in that case the
  article says the MS1annotation() function should be used instead of MSMSannotation().
- When the feature table has not yet been through isotope pattern mapping or adduct
  computation, since MSMSannotation expects those steps as prerequisite inputs.
edam_operation: http://edamontology.org/operation_3795
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ipaPy2
  role: Python library whose MSMSannotation() function performs the cosine-similarity
    scoring of measured MS2 spectra against database spectra and reintegration into
    posterior probabilities.
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Loads the feature table, adducts table, MS2 spectra CSV and MS2 database CSV
    into dataframes for annotation.
- name: numpy
  role: Numerical backend used alongside pandas for the dataframe operations in the
    tutorial.
- name: PeakMLViewerPy
  role: Graphical viewer for exploring the annotation results produced by ipaPy2.
  repo: https://github.com/UoMMIB/PeakMLViewerPy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:cc9b7f1bdc6f4dcd31449a725d3b7cb68859d67752b0f7885c1fc1c61b998e1d
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
  edam_operation: http://edamontology.org/operation_3795
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# collision-energy-spectrum-handling

## Summary

Handling of MS2 fragmentation spectra measured at one or more collision energies within the ipaPy2 MSMSannotation() scoring step: measured spectra are compared against database spectra by cosine similarity, with the highest-scoring match supplying the fragmentation pattern score used in the Bayesian posterior.

## When to use

Use when a feature table already has isotope-mapped entries and measured MS2 spectra (e.g. MS2data_example.csv) plus an MS2 database (DBMS2_test_pos.csv) are available, and you want to score candidate annotations by fragmentation pattern similarity.

## When NOT to use

- When no fragmentation data has been acquired for the features — in that case the article says the MS1annotation() function should be used instead of MSMSannotation().
- When the feature table has not yet been through isotope pattern mapping or adduct computation, since MSMSannotation expects those steps as prerequisite inputs.

## Inputs

- Isotope-pattern-mapped feature table (pandas dataframe, e.g. ExampleDatasets/README/df_test_pos.csv)
- Adducts table (pandas dataframe, output of compute_all_adducts())
- Measured MS2 spectra CSV (ExampleDatasets/README/MS2data_example.csv)
- MS2 database CSV (DB/DBMS2_test_pos.csv)
- Adducts reference file (DB/adducts.csv)

## Outputs

- Dictionary of per-feature annotation dataframes containing a 'fragmentation pattern score' column and updated 'post' probabilities
- Pickled annotations dictionary (e.g. annotations_msms.pkl)

## How to apply

Load the isotope-mapped positive-mode feature table, the adducts table produced by compute_all_adducts(), the MS2 spectra (ExampleDatasets/README/MS2data_example.csv) and the reduced MS2 database (DB/DBMS2_test_pos.csv) with pandas. Call ipa.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1). For every candidate annotation the measured spectrum is matched against database spectra with a default ppmCS=10 fragment tolerance; by default evfilt=False so all collision energies present are pooled rather than filtered. The cosine similarity is computed for each (measured, database) spectrum pair and the highest value is taken as the 'fragmentation pattern score'. Candidates for which no database spectrum exists receive the CSunk=0.7 dummy score so they stay comparable to scored candidates. Finally the posterior probabilities of each annotation are recomputed by integrating this fragmentation pattern score with mass accuracy, RT range, isotope pattern score and prior knowledge, and the returned per-feature annotation dataframes should be pickled (e.g. annotations_msms.pkl).

## Related tools

- **ipaPy2** (Python library whose MSMSannotation() function performs the cosine-similarity scoring of measured MS2 spectra against database spectra and reintegration into posterior probabilities.) — https://github.com/francescodc87/ipaPy2
- **pandas** (Loads the feature table, adducts table, MS2 spectra CSV and MS2 database CSV into dataframes for annotation.)
- **numpy** (Numerical backend used alongside pandas for the dataframe operations in the tutorial.)
- **PeakMLViewerPy** (Graphical viewer for exploring the annotation results produced by ipaPy2.) — https://github.com/UoMMIB/PeakMLViewerPy

## Examples

```
ipa.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1)
```

## Evaluation signals

- The returned dictionary contains for each annotated feature a 'fragmentation pattern score' column populated with cosine similarity values (verifiable for e.g. feature id=1 as in the tutorial).
- Each candidate annotation with no corresponding database MS2 spectrum carries the CSunk=0.7 dummy score rather than a computed cosine similarity.
- The 'post' probability column differs from the MS1-only posterior and reflects integration of mass accuracy, RT range, isotope pattern score, prior knowledge and the fragmentation pattern score.
- Running the invocation with ncores=1 reproduces the tutorial output without errors; pickle file annotations_msms.pkl loads cleanly.

## Limitations

- Default evfilt=False means all collision energies are pooled; if collision-energy filtering is required, the user must change that parameter explicitly.
- Candidate annotations lacking a database MS2 spectrum are penalised only by the CSunk=0.7 dummy score, not by a real measured-vs-database comparison.
- The ppmCS fragment matching tolerance defaults to 10 ppm; changing it alters which database fragments are considered matches and thus the cosine similarity.
- The article recommends building an ad-hoc MS1 database matched to instrument setup and sample type; a drastically reduced example database (DBMS2_test_pos.csv) is included only for illustration.

## Evidence

- [methods] MS2 integration scoring: [evidence span withheld — anchor status: not_found]
- [readme] evfilt default: "fragmentation data can be included in the annotation process by using the MSMSannotation() function"
- [methods] example files: "dfMS2=pd.read_csv('ExampleDatasets/README/MS2data_example.csv')"
- [methods] no fragmentation data case: "If no fragmentation data is available, the MS1annotation() function should be used"
- [abstract] integration into posterior: "The revised method is now able to integrate tandem MS fragmentation data, which increases the accuracy of the identifications."
