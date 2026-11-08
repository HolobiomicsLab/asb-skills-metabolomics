---
name: tandem-ms-spectral-cosine-similarity-scoring
description: Use when LC–MS/MS untargeted metabolomics features already have isotope-pattern-mapped
  adducts and fragmentation data (MS2 spectra) are available, so candidate compounds
  can be scored against their stored reference spectra.
when_to_use_negative:
- No fragmentation (MS2) data was acquired — use MS1annotation() instead, which scores
  candidates from mass accuracy, RT range, isotope pattern and prior knowledge alone.
- The adducts table has not yet been computed for the database — the pipeline requires
  compute_all_adducts() output before MSMSannotation() can be invoked.
- The feature table has not been clustered/isotope-pattern-mapped (map_isotope_patterns()
  / clusterFeatures() not yet run) — the input dataframe must already contain mapped
  isotope fingerprints.
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ipaPy2
  role: Provides the MSMSannotation() function that performs the MS2-vs-database cosine
    similarity scoring and integrates the resulting fragmentation pattern score into
    posterior probability computation.
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Used to load the packaged positive-mode feature table, adducts table, MS2
    data and reduced MS2 database as dataframes before calling MSMSannotation().
- name: PeakMLViewerPy
  role: GUI tool for exploring the results of the annotation, including the MS2-based
    fragmentation pattern scores.
  repo: https://github.com/UoMMIB/PeakMLViewerPy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:835f144b7dc94fdcbb79caff0059c8e7b0e9acd51c83486221f1e62519898b86
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# tandem-ms-spectral-cosine-similarity-scoring

## Summary

Scores candidate MS1 annotations by comparing acquired MS2 fragmentation spectra against a database of reference spectra, using cosine similarity to produce a 'fragmentation pattern score' that is integrated into IPA posterior probability computation.

## When to use

Use when LC–MS/MS untargeted metabolomics features already have isotope-pattern-mapped adducts and fragmentation data (MS2 spectra) are available, so candidate compounds can be scored against their stored reference spectra.

## When NOT to use

- No fragmentation (MS2) data was acquired — use MS1annotation() instead, which scores candidates from mass accuracy, RT range, isotope pattern and prior knowledge alone.
- The adducts table has not yet been computed for the database — the pipeline requires compute_all_adducts() output before MSMSannotation() can be invoked.
- The feature table has not been clustered/isotope-pattern-mapped (map_isotope_patterns() / clusterFeatures() not yet run) — the input dataframe must already contain mapped isotope fingerprints.

## Inputs

- Isotope-pattern-mapped feature table dataframe (e.g., ExampleDatasets/README/df_test_pos.csv with 'id' and intensities across samples)
- Adducts table produced by compute_all_adducts() (allAddsPos for positive mode)
- Measured MS2 spectra dataframe (e.g., ExampleDatasets/README/MS2data_example.csv)
- MS2 reference spectra database (e.g., DB/DBMS2_test_pos.csv)
- MS1 database file (required schema: id, name, formula, inchi, smiles, RT, adductsPos, adductsNeg, description, pk, MS2, reactions)

## Outputs

- Dictionary of per-feature annotation dataframes, each containing a 'fragmentation pattern score' column populated with cosine similarity values
- Updated 'post' posterior probabilities per candidate annotation, integrating fragmentation pattern score with mass accuracy, RT range, isotope pattern score, and prior knowledge

## How to apply

Load the isotope-pattern-mapped positive-mode feature table (e.g., ExampleDatasets/README/df_test_pos.csv), the adducts table from compute_all_adducts(), the measured MS2 data (ExampleDatasets/README/MS2data_example.csv), and a reduced MS2 database (DB/DBMS2_test_pos.csv) with pandas. Call ipa.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1). For each feature with acquired fragmentation data, the measured MS2 spectra are compared against the spectra stored in the database for that annotation and the highest cosine similarity score is taken as the fragmentation pattern score; the default fragment matching tolerance is ppmCS=10, and evfilt=False means all collision energies are used. Candidate annotations with no fragmentation spectrum in the database receive the dummy CSunk=0.7 cosine score so they remain comparable to scored candidates. The returned dictionary of per-feature annotation dataframes has the 'fragmentation pattern score' column populated with cosine similarity values and recomputed 'post' posterior probabilities that integrate mass accuracy, RT range, isotope pattern score, prior knowledge, and the new fragmentation score.

## Related tools

- **ipaPy2** (Provides the MSMSannotation() function that performs the MS2-vs-database cosine similarity scoring and integrates the resulting fragmentation pattern score into posterior probability computation.) — https://github.com/francescodc87/ipaPy2
- **pandas** (Used to load the packaged positive-mode feature table, adducts table, MS2 data and reduced MS2 database as dataframes before calling MSMSannotation().)
- **PeakMLViewerPy** (GUI tool for exploring the results of the annotation, including the MS2-based fragmentation pattern scores.) — https://github.com/UoMMIB/PeakMLViewerPy

## Examples

```
ipa.MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1)
```

## Evaluation signals

- The script runs without error and returns a pickled dictionary (annotations_msms.pkl) whose keys are feature ids and values are annotation dataframes.
- For each annotated feature with acquired fragmentation data, the 'fragmentation pattern score' column contains cosine similarity values (verifiable, e.g., for feature id=1 as in the tutorial).
- Candidate annotations with no fragmentation spectrum in the database carry the CSunk=0.7 dummy score, distinguishing them from scored candidates.
- Posterior 'post' probabilities are recomputed and are consistent with the integration of the fragmentation pattern score with mass accuracy, RT range, isotope pattern score and prior knowledge.
- Entries in the MS2 database (e.g., DBMS2_test_pos.csv) match those referenced by the MS1 database 'MS2' id column, so candidate-to-reference spectra linkage is non-empty.

## Limitations

- The method is only as good as the MS2 database: for Q-Exactive-type instruments MoNa is suggested, and if no fragmentation spectrum exists for a candidate, a dummy CSunk=0.7 score is assigned rather than a true match.
- Users are strongly advised to build their own ad-hoc database to match their specific instrument setup and sample types; the provided IPA_MS1.csv is only a relatively big example.
- If an exotic adduct (or in-source fragment) needs to be considered, the user must modify the provided adducts.csv file accordingly.

## Evidence

- [methods] MSMSannotation invocation and scoring rule: "fragmentation data can be included in the annotation process by using the MSMSannotation() function"
- [results] Cosine similarity scoring of measured vs database MS2 spectra: [evidence span withheld — anchor status: not_found]
- [abstract] Fragmentation integration increases identification accuracy: "The revised method is now able to integrate tandem MS fragmentation data, which increases the accuracy of the identifications."
- [readme] MSMSannotation with CSunk and ppm parameters from tutorial: "dfMS2=pd.read_csv('ExampleDatasets/README/MS2data_example.csv')"
- [methods] Reduced example MS2 database provided: "a drastically reduced version of it has been included within this repository, and can be found [here](DB/DBMS2_test_pos.csv)"
