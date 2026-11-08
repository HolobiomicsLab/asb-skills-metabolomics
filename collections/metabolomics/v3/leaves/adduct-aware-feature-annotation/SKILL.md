---
name: adduct-aware-feature-annotation
description: Use when annotating LC–MS untargeted metabolomics features against a
  compound database in either positive or negative ionisation mode, and candidate
  metabolites must be scored accounting for the specific adducts they can form.
when_to_use_negative:
- You already have a curated per-compound adduct list and only need re-scoring — building
  the all-adducts table from adducts.csv again is redundant
- The analysis concerns MS/MS spectra comparison only, with no MS1-level adduct enumeration
  required (MSMSannotation on pre-computed allAdds suffices)
- Compounds carry exotic adducts or in-source fragments not in DB/adducts.csv and
  the adduct file has not been modified accordingly
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: ipaPy2
  role: Provides compute_all_adducts(), map_isotope_patterns(), clusterFeatures(),
    MS1annotation() and MSMSannotation() that together implement the adduct-aware
    annotation pipeline
  repo: https://github.com/francescodc87/ipaPy2
- name: PeakMLViewerPy
  role: Standalone viewer for exploring ipaPy2 annotation results
  repo: https://github.com/UoMMIB/PeakMLViewerPy
- name: mzMatch
  role: Pre-processing pipeline whose output feature tables feed into ipaPy2 annotation
  repo: https://github.com/UoMMIB/mzmatch.R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:2a7f65269ffc287a6ff8dbe4b58ffbf6a899acfdf8843103d4dc86351ebd155e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - PeakMLViewerPy
  - Python
  - ipaPy2
  - mzMatch
  - numpy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Adduct-aware feature annotation

## Summary

Computes all possible adducts a database compound can form per ionisation mode and maps them to observed LC–MS features so that isotope fingerprints, mass accuracy, RT ranges and prior knowledge are combined into posterior annotation probabilities using ipaPy2's compute_all_adducts().

## When to use

Use when annotating LC–MS untargeted metabolomics features against a compound database in either positive or negative ionisation mode, and candidate metabolites must be scored accounting for the specific adducts they can form.

## When NOT to use

- You already have a curated per-compound adduct list and only need re-scoring — building the all-adducts table from adducts.csv again is redundant
- The analysis concerns MS/MS spectra comparison only, with no MS1-level adduct enumeration required (MSMSannotation on pre-computed allAdds suffices)
- Compounds carry exotic adducts or in-source fragments not in DB/adducts.csv and the adduct file has not been modified accordingly

## Inputs

- MS1 database dataframe with columns id, name, formula, inchi, smiles, RT, adductsPos, adductsNeg, description, pk, MS2, reactions (e.g. DB/IPA_MS1.csv)
- Adducts definition table (DB/adducts.csv) with columns name, calc, Charge, Mult, Mass, Ion_mode, Formula_add, Formula_ded, Multi
- Feature table with measured intensities across samples, ideally already isotope-pattern-mapped (e.g. ExampleDatasets/README/df_test_pos.csv)

## Outputs

- All-adducts table enumerating possible adducts per database compound per ionisation mode (allAddsPos/allAddsNeg)
- Annotated feature dataframe with per-annotation posterior probabilities ('post' column), isotope pattern score and, when fragmentation is used, fragmentation pattern score

## How to apply

First prepare a MS1 database dataframe with the exact columns required by ipaPy2 (id, name, formula, inchi, smiles, RT, adductsPos, adductsNeg, description, pk, MS2, reactions) and load the adducts.csv file shipped with the package, which lists common adducts such as M+H, M+NH4, M+Na and M+K in positive mode and M-H, M-2H in negative mode. Call ipa.compute_all_adducts(DB, adducts) to enumerate every possible adduct per database entry per ionisation mode, producing an all-adducts table keyed to each compound's formula and charge state. If the feature table has not yet been isotope-mapped, first run map_isotope_patterns() (and optionally clusterFeatures()) so that isotope peaks are integrated into isotope fingerprints rather than treated as individual features, which speeds up calculations. Then pass the mapped feature dataframe and the all-adducts table into MS1annotation() or MSMSannotation(), where mass accuracy (ppm tolerance), retention time range, isotope pattern score and prior knowledge (pk) are combined to recompute posterior probabilities per annotation. Optionally include previous-experiment knowledge by setting the pk column between 0 (compound cannot be present) and 1 (highly likely to be present).

## Related tools

- **ipaPy2** (Provides compute_all_adducts(), map_isotope_patterns(), clusterFeatures(), MS1annotation() and MSMSannotation() that together implement the adduct-aware annotation pipeline) — https://github.com/francescodc87/ipaPy2
- **PeakMLViewerPy** (Standalone viewer for exploring ipaPy2 annotation results) — https://github.com/UoMMIB/PeakMLViewerPy
- **mzMatch** (Pre-processing pipeline whose output feature tables feed into ipaPy2 annotation) — https://github.com/UoMMIB/mzmatch.R

## Examples

```
import ipaPy2 as ipa; import pandas as pd; adducts=pd.read_csv('DB/adducts.csv'); DB=pd.read_csv('DB/IPA_MS1.csv'); allAddsPos=ipa.compute_all_adducts(DB, adducts)
```

## Evaluation signals

- The all-adducts table contains exactly the expected entries for each database compound's adductsPos/adductsNeg lists and Ion_mode column
- Every annotated feature carries a posterior probability ('post') between 0 and 1 and an isotope pattern score
- Features with no valid adduct candidate remain unannotated rather than receiving a spurious annotation
- Pipeline completes without error on the packaged test data (df_test_pos.csv + DB_test_pos.csv) reproducing tutorial behaviour
- When run with MS2 data, 'fragmentation pattern score' is populated with cosine similarity values (e.g. for feature id=1 as in the tutorial)

## Limitations

- Users must build their own ad-hoc database matching their instrument setup and sample types; the packaged IPA_MS1.csv is only an example
- Exotic adducts or in-source fragments require manual modification of the adducts.csv file
- ipaPy2 requires Python 3.9 or higher
- Isotope peaks must be integrated into isotope fingerprints before annotation; running on unclustered, un-mapped tables (df_test_pos_not_clustered.csv) will not work as intended

## Evidence

- [readme] Adducts file is required input defining all possible adducts per ionisation mode: "The ipaPy2 library requires a file contains all the information required for the computation of the adducts. An adducts.csv file is provided with the package"
- [readme] MS1 database dataframe must have exact column names and order: "This dataframe must contain the following columns in this exact order (optional columns can have empty fields)"
- [abstract] Isotope peaks integrated into fingerprints for speed: "isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations."
- [methods] compute_all_adducts is the second pipeline step: "The second step of the pipeline consists in the calculation of all possible adducts that could be formed by the compounds included in the database. This is done by the function compute_all_adducts()"
- [readme] pk prior knowledge scale 0–1: "The value has to be between 1 (compound highly likely to be present in the sample) and 0 (compound cannot be present in the sample)."
- [abstract] Annotation integrates mass accuracy, RT, isotope pattern and prior knowledge: "provides statistically rigorous estimates of the probabilities"
