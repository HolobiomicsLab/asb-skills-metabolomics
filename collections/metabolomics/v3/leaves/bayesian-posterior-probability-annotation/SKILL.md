---
name: bayesian-posterior-probability-annotation
description: Use when you have a clustered MS1 feature table (or MS1+MS2 spectra)
  and a candidate compound database with adducts, and you need probability-ranked
  metabolite annotations rather than a single match.
when_to_use_negative:
- Input features are not yet clustered — first apply ipaPy2's clusterFeatures() on
  the unclustered intensity dataframe (e.g. df_test_pos_not_clustered.csv), since
  the annotation expects isotope-fingerprinted feature groups
- Only a single best compound identity is needed with no probability ranking — the
  IPA method is designed to return posterior probability distributions over all candidate
  annotations, not a single deterministic label
- No compound database with adduct lists (adductsPos/adductsNeg) is available — the
  method cannot enumerate candidate annotations without a properly formatted DB file
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: ipaPy2
  role: Python implementation of the IPA Bayesian annotation method; provides map_isotope_patterns(),
    compute_all_adducts(), MS1annotation() and MSMSannotation() to produce posterior
    probabilities
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Loads and structures the adducts file, database and feature table required
    by the annotation functions
- name: numpy
  role: Numerical support for probability calculations on the feature and annotation
    dataframes
- name: PeakMLViewerPy
  role: Explores and visualises the annotation results produced by ipaPy2
  repo: https://github.com/UoMMIB/PeakMLViewerPy
- name: mzMatch
  role: Downstream pipeline with which ipaPy2 annotation results are fully integrated
  repo: https://github.com/UoMMIB/mzmatch.R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:92b7ad7e756fcec721c9ae50669628fbf9333c5de0aec88f4f25f0e11d4eba4d
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
  - pandas
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# bayesian-posterior-probability-annotation

## Summary

Bayesian posterior probability annotation of LC-MS untargeted metabolomics features using ipaPy2's IPA method, which integrates mass accuracy, isotope patterns, adduct formation, retention time ranges and prior knowledge to output statistically rigorous posterior probabilities per annotation.

## When to use

Use when you have a clustered MS1 feature table (or MS1+MS2 spectra) and a candidate compound database with adducts, and you need probability-ranked metabolite annotations rather than a single best match.

## When NOT to use

- Input features are not yet clustered — first apply ipaPy2's clusterFeatures() on the unclustered intensity dataframe (e.g. df_test_pos_not_clustered.csv), since the annotation expects isotope-fingerprinted feature groups
- Only a single best compound identity is needed with no probability ranking — the IPA method is designed to return posterior probability distributions over all candidate annotations, not a single deterministic label
- No compound database with adduct lists (adductsPos/adductsNeg) is available — the method cannot enumerate candidate annotations without a properly formatted DB file

## Inputs

- Clustered MS1 feature dataframe (pandas) with measured intensities across samples, e.g. df_test_pos.csv
- Adducts CSV file (DB/adducts.csv) with columns name, calc, Charge, Mult, Mass, Ion_mode, Formula_add, Formula_ded, Multi
- MS1 compound database (e.g. DB/DB_test_pos.csv or DB/IPA_MS1.csv) with columns id, name, formula, inchi, smiles, RT, adductsPos, adductsNeg, description, pk, MS2, reactions
- Optional MS2 spectra dataframe (e.g. ExampleDatasets/README/MS2data_example.csv) for MSMSannotation()

## Outputs

- Annotations dictionary keyed by feature id, each entry a dataframe with columns: id, name, formula, adduct, m/z, charge, RT range, ppm, isotope pattern score, fragmentation pattern score, prior, post
- ms1_annotations.csv containing per-feature annotation rows with populated 'prior' and 'post' probability columns

## How to apply

Load the clustered feature dataframe (e.g. ExampleDatasets/README/df_test_pos.csv), the packaged adducts file (DB/adducts.csv) and the database (DB/DB_test_pos.csv) with pandas. Run ipa.map_isotope_patterns(df, ionisation=1) to collapse isotope peaks into isotope fingerprints and assign charges, then ipa.compute_all_adducts(adducts, DB, ionisation=1, ncores=1) to enumerate all possible adducts for each database compound. For MS1-only annotation call ipa.MS1annotation(df, allAddsPos, ppm=3, ncores=1) with default priors (me=0.000548579909065, ratiosd=0.9, pRTNone=0.8, pRTout=0.4); the function first computes prior probabilities from mass alone and then uses a Gibbs sampler (step 2 of the IPA method) to estimate posterior probabilities by considering all possible biochemical connections and connections between adducts. If MS2 spectra are available, use ipa.MSMSannotation() instead to integrate fragmentation data and increase annotation accuracy. The result is an annotations dictionary keyed by feature id, each holding a dataframe with id, name, formula, adduct, m/z, charge, RT range, ppm, isotope pattern score, fragmentation pattern score, prior and post columns, which you write out as ms1_annotations.csv.

## Related tools

- **ipaPy2** (Python implementation of the IPA Bayesian annotation method; provides map_isotope_patterns(), compute_all_adducts(), MS1annotation() and MSMSannotation() to produce posterior probabilities) — https://github.com/francescodc87/ipaPy2
- **pandas** (Loads and structures the adducts file, database and feature table required by the annotation functions)
- **numpy** (Numerical support for probability calculations on the feature and annotation dataframes)
- **PeakMLViewerPy** (Explores and visualises the annotation results produced by ipaPy2) — https://github.com/UoMMIB/PeakMLViewerPy
- **mzMatch** (Downstream pipeline with which ipaPy2 annotation results are fully integrated) — https://github.com/UoMMIB/mzmatch.R

## Examples

```
ipa.MS1annotation(df, allAddsPos, ppm=3, ncores=1)
```

## Evaluation signals

- ms1_annotations.csv exists and is parseable as a CSV
- Each per-feature annotation row carries non-empty 'prior' and 'post' probability columns
- Feature id=1 is annotated with L-Proline M+H at the highest posterior probability, matching the tutorial's expected result
- The annotations dictionary is keyed by feature id and each entry is a dataframe with the documented columns (id, name, formula, adduct, m/z, charge, RT range, ppm, isotope pattern score, fragmentation pattern score, prior, post)
- Posterior values are within [0,1] and generally differ from prior values where biochemical connections or isotope patterns shift the estimate

## Limitations

- ipaPy2 requires Python 3.9 or higher
- The adducts file must be modified by hand if exotic adducts or in-source fragments need to be considered
- Without MS2 data, annotation relies solely on MS1 evidence; the MSMSannotation() function is required to integrate fragmentation data for higher accuracy
- Database entries must follow the exact column order and names specified (id, name, formula, inchi, smiles, RT, adductsPos, adductsNeg, description, pk, MS2, reactions); optional columns may be empty but required ones must be present

## Evidence

- [methods] Gibbs sampler estimates posterior probabilities of all annotations: [evidence span withheld — anchor status: not_found]
- [methods] MS1annotation used without fragmentation data: "If no fragmentation data is available, the MS1annotation() function should be used"
- [abstract] MS2 integration increases accuracy: "The revised method is now able to integrate tandem MS fragmentation data, which increases the accuracy of the identifications."
- [abstract] Isotope fingerprints speed up calculations: "isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations."
- [abstract] Statistically rigorous probability estimates: "The Integrated Probabilistic Annotation (IPA) is an automated annotation method for LC-MS-based untargeted metabolomics experiments that provides statistically rigorous estimates of the probabilities"
- [readme] Python 3.9+ requirement: "ipaPy2 requires Python 3.9 or higher"
- [methods] MSMSannotation function: "fragmentation data can be included in the annotation process by using the MSMSannotation() function"
- [methods] Compute all possible adducts: "The second step of the pipeline consists in the calculation of all possible adducts that could be formed by the compounds included in the database. This is done by the function compute_all_adducts()"
- [readme] Database column requirements: "This dataframe must contain the following columns in this exact order (optional columns can have empty fields): **id**: unique id of the database entry"
