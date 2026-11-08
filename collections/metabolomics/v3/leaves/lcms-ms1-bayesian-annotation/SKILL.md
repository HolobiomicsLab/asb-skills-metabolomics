---
name: lcms-ms1-bayesian-annotation
description: Use when a clustered MS1 feature table (intensities across samples) from
  LC-MS untargeted metabolomics needs compound and adduct annotations with statistically
  rigorous probability estimates, especially when MS2 spectra or prior experiment
  knowledge should be integrated.
when_to_use_negative:
- Input is raw mzML or other instrument output that has not yet been peak-picked and
  clustered (the 'clusterFeatures()' step) — pre-process with mzMatch/CAMERA-compatible
  pipelines first
- The dataset contains no MS2 spectra and you want fragmentation-evidence-based scores
  — MSMSannotation requires an MS2 database (e.g. DBMS2_test_pos.csv) and only adds
  accuracy when such data exist
- A customised compound database is inappropriate for the instrument setup and sample
  types — users are strongly advised to build their own ad-hoc database; off-the-shelf
  DB/IPA_MS1.csv may not match
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: ipaPy2
  role: Python implementation of the Integrated Probabilistic Annotation (IPA) 2.0,
    providing map_isotope_patterns, compute_all_adducts, MS1annotation and MSMSannotation
    functions
  repo: https://github.com/francescodc87/ipaPy2
- name: PeakMLViewerPy
  role: Viewer to explore the results of the annotation visually
  repo: https://github.com/UoMMIB/PeakMLViewerPy
- name: pandas
  role: Loads the feature table, MS2 data, compound databases and adducts file as
    dataframes
- name: mzMatch
  role: Recommended upstream data processing software for generating feature tables
    that ipaPy2 is fully integrated with
  repo: https://github.com/UoMMIB/mzmatch.R
- name: CAMERA
  role: Recommended upstream data processing software for feature clustering
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:0990add8697cdb85d319879497bcd9693a865b19232917561ca10b37837a612b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - CAMERA
  - PeakMLViewerPy
  - Python
  - ipaPy2
  - mzMatch
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# lcms-ms1-bayesian-annotation

## Summary

Statistically rigorous Bayesian annotation of LC–MS/MS untargeted metabolomics features via ipaPy2, integrating isotope patterns, adduct formation, biochemical relations, and tandem MS fragmentation to yield per-annotation prior and posterior probabilities.

## When to use

Use when a clustered MS1 feature table (intensities across samples) from LC-MS untargeted metabolomics needs compound and adduct annotations with statistically rigorous probability estimates, especially when MS2 spectra or prior experiment knowledge should be integrated.

## When NOT to use

- Input is raw mzML or other instrument output that has not yet been peak-picked and clustered (the 'clusterFeatures()' step) — pre-process with mzMatch/CAMERA-compatible pipelines first
- The dataset contains no MS2 spectra and you want fragmentation-evidence-based scores — MSMSannotation requires an MS2 database (e.g. DBMS2_test_pos.csv) and only adds accuracy when such data exist
- A customised compound database is inappropriate for the instrument setup and sample types — users are strongly advised to build their own ad-hoc database; off-the-shelf DB/IPA_MS1.csv may not match

## Inputs

- Clustered MS1 feature table (pandas dataframe of measured intensities across samples), e.g. ExampleDatasets/README/df_test_pos.csv
- MS2 spectral peaks dataframe, e.g. ExampleDatasets/README/MS2data_example.csv
- MS1 compound database with columns id, name, formula, inchi, smiles, RT, adductsPos, adductsNeg, description, pk, MS2, reactions (e.g. DB/IPA_MS1.csv or DB/DB_test_pos.csv)
- MS2 database of fragmentation spectra, e.g. DB/DBMS2_test_pos.csv
- Adducts table DB/adducts.csv with name, calc, Charge, Mult, Mass, Ion_mode, Formula_add, Formula_ded, Multi

## Outputs

- Per-feature annotation dictionary mapping (compound id, adduct) to 'prior' and 'post' probabilities
- Adduct-annotated isotope-pattern fingerprints with predicted charges
- All-adducts candidate table for the compound database
- Optional new Peaks.csv file loadable in PeakMLViewerPy for visual exploration of annotations

## How to apply

Run the ipaPy2 pipeline in Python 3.9+: first map isotope patterns with map_isotope_patterns(df, ionisation=1) so isotope peaks become fingerprints rather than individual features, then compute candidate adducts with compute_all_adducts(adducts, DB, ionisation=1, ncores=1) using the packaged DB/adducts.csv. Annotate against a compound database (e.g. KEGG or Natural Products Atlas derived) using MS1annotation(df, allAddsPos, ppm=3, ncores=1) when no fragmentation data exist, or MSMSannotation(df, dfMS2, allAddsPos, DBMS2, CSunk=0.7, ppm=3, ncores=1) when MS2 spectra are available — the latter adds a fragmentation pattern cosine-similarity score, with CSunk=0.7 penalising unmatched MS2 peaks. Integrate prior experiment knowledge through the 'pk' column of the database (1 = highly likely present, 0 = cannot be present) and biochemical connections via the 'reactions' column; a Gibbs sampler recomputes annotations iteratively so runtime grows non-linearly with feature count. Compare prior and posterior probabilities per (compound id, adduct) annotation to rank identifications, and inspect results in PeakMLViewerPy.

## Related tools

- **ipaPy2** (Python implementation of the Integrated Probabilistic Annotation (IPA) 2.0, providing map_isotope_patterns, compute_all_adducts, MS1annotation and MSMSannotation functions) — https://github.com/francescodc87/ipaPy2
- **PeakMLViewerPy** (Viewer to explore the results of the annotation visually) — https://github.com/UoMMIB/PeakMLViewerPy
- **pandas** (Loads the feature table, MS2 data, compound databases and adducts file as dataframes)
- **mzMatch** (Recommended upstream data processing software for generating feature tables that ipaPy2 is fully integrated with) — https://github.com/UoMMIB/mzmatch.R
- **CAMERA** (Recommended upstream data processing software for feature clustering)

## Examples

```
python -c "import pandas as pd, ipaPy2 as ipa; df=pd.read_csv('ExampleDatasets/README/df_test_pos.csv'); df=ipa.map_isotope_patterns(df, ionisation=1); allAddsPos=ipa.compute_all_adducts(pd.read_csv('DB/adducts.csv'), pd.read_csv('DB/DB_test_pos.csv'), ionisation=1, ncores=1); res=ipa.MS1annotation(df, allAddsPos, ppm=3, ncores=1)"
```

## Evaluation signals

- Every annotation row carries both 'prior' and 'post' probability fields — a run missing either is incomplete
- Feature-level output contains assignment of relationships (bp, bp|isotope, potential bp), isotope patterns and predicted charges after map_isotope_patterns
- Annotations for features with acquired MS2 spectra must carry non-empty fragmentation pattern (cosine similarity) scores when MSMSannotation is used
- Runtime per Gibbs sampler iteration increases monotonically and non-linearly with the number of annotated features, as reported in Supplementary Fig. S1
- Setting 'pk' to 0 for compounds that cannot be present removes them from posterior annotations (probability near 0)

## Limitations

- Gibbs sampler computational time per iteration increases non-linearly with feature count, so very large feature sets are slow
- The MS2 database used for ipaPy2 example data was restricted to MoNa entries acquired with a Thermo Q-Exactive Plus; only compounds with at least one such spectrum can be scored
- Custom or exotic adducts and in-source fragments require manual editing of DB/adducts.csv
- Users are strongly advised to build their own ad-hoc database matching their specific instrument setup and sample types

## Evidence

- [intro] Bayesian annotation integrating isotope, adduct and biochemical relations: "Python implementation of the Integrated Probabilistic Annotation (IPA) - A Bayesian annotation method for LC/MS data integrating biochemical relations, isotope patterns and adduct formation."
- [abstract] MS2 integration increases accuracy: "The revised method is now able to integrate tandem MS fragmentation data, which increases the accuracy of the identifications."
- [methods] MS1annotation and MSMSannotation functions: "If no fragmentation data is available, the MS1annotation() function should be used"
- [abstract] isotope fingerprints speed-up: "isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations."
- [readme] install via pip: "### Install via pip (recommended)

```
pip install ipaPy2
```"
