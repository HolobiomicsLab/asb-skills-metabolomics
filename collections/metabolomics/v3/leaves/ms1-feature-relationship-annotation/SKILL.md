---
name: ms1-feature-relationship-annotation
description: Use when you have a clustered MS1 feature table (pandas dataframe with
  ids, rel.ids, mzs, RTs, Int columns) and need to collapse isotope peaks into isotope
  pattern units before adduct computation or IPA annotation.
when_to_use_negative:
- When the input dataframe is not yet clustered — run clusterFeatures() first, e.g.
  on ExampleDatasets/README/df_test_pos_not_clustered.csv
- When fragmentation (MS2) data is available and you intend to use MSMSannotation();
  this isotope mapping alone does not perform annotation
- When the goal is adduct calculation rather than isotope-pattern grouping — use compute_all_adducts()
  instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: ipaPy2
  role: Provides map_isotope_patterns() which performs the isotope-pattern mapping
    on the clustered MS1 feature dataframe
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Used to load the clustered feature table (pd.read_csv) and to hold and save
    the modified dataframe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:a47ae2b6c92fce1f96981d90d058cc4c29c9f74dd327acf32cc33ab94c032c26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - Python
  - ipaPy2
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ms1-feature-relationship-annotation

## Summary

Maps isotope patterns within a clustered MS1 feature table using ipaPy2's map_isotope_patterns(), so that isotope peaks are grouped into per-metabolite isotope fingerprints rather than treated as individual features upstream of Bayesian annotation.

## When to use

Use when you have a clustered MS1 feature table (pandas dataframe with ids, rel.ids, mzs, RTs, Int columns) and need to collapse isotope peaks into isotope pattern units before adduct computation or IPA annotation.

## When NOT to use

- When the input dataframe is not yet clustered — run clusterFeatures() first, e.g. on ExampleDatasets/README/df_test_pos_not_clustered.csv
- When fragmentation (MS2) data is available and you intend to use MSMSannotation(); this isotope mapping alone does not perform annotation
- When the goal is adduct calculation rather than isotope-pattern grouping — use compute_all_adducts() instead

## Inputs

- Clustered MS1 feature table as a pandas dataframe (e.g. ExampleDatasets/README/df_test_pos.csv) with ids, rel.ids, mzs, RTs and Int columns
- Ionisation mode as an integer (1 for positive, -1 for negative)

## Outputs

- Same dataframe modified in place with additional relationship, isotope pattern and charge columns
- Annotated isotope pattern table saved as CSV (isotope_patterns.csv)

## How to apply

Load the clustered MS1 feature table (e.g. ExampleDatasets/README/df_test_pos.csv) into a pandas dataframe; it must already contain ids, rel.ids, mzs, RTs and Int columns. Import the ipaPy2.ipa module and call ipa.map_isotope_patterns(df, ionisation=1) for positive mode, relying on the function defaults isoDiff=1, ppm=100 and MinIsoRatio=0.5; use ionisation=-1 for negative mode data. The function modifies the dataframe in place, adding and populating relationship, isotope pattern and charge columns that link each feature to its base peak. This step is the first of the IPA pipeline and is deliberately upstream of compute_all_adducts() and annotation, because treating isotope peaks as a single unit greatly speeds up the subsequent Bayesian calculations and avoids double-counting isotopes as separate metabolites. Save the annotated dataframe (e.g. as isotope_patterns.csv) and validate the relationship labels and charge range before proceeding to adduct computation.

## Related tools

- **ipaPy2** (Provides map_isotope_patterns() which performs the isotope-pattern mapping on the clustered MS1 feature dataframe) — https://github.com/francescodc87/ipaPy2
- **pandas** (Used to load the clustered feature table (pd.read_csv) and to hold and save the modified dataframe)

## Examples

```
import pandas as pd; from ipaPy2 import ipa; df = pd.read_csv('ExampleDatasets/README/df_test_pos.csv'); df = ipa.map_isotope_patterns(df, ionisation=1); df.to_csv('isotope_patterns.csv')
```

## Evaluation signals

- isotope_patterns.csv exists after the step and the relationship column contains only the allowed labels: bp, bp|isotope, potential bp, potential bp|isotope
- Charge column values all fall within the expected range of -5 to 5
- The dataframe retains its original ids/rel.ids/mzs/RTs/Int columns and gains relationship, isotope-pattern and charge columns in place
- Downstream compute_all_adducts() or annotation runs without errors on the mapped dataframe, indicating isotope units were accepted
- No isotope peaks appear as standalone entries in the annotation input — each isotope is tied to a base-peak relationship label

## Limitations

- Defaults isoDiff=1, ppm=100, MinIsoRatio=0.5 are used unless overridden; unusual adducts or in-source fragments require modifying the adducts file rather than these parameters
- Exotic adducts or in-source fragments need the user to modify DB/adducts.csv, not this mapping step
- Requires the input dataframe to be pre-clustered; unclustered tables (e.g. df_test_pos_not_clustered.csv) are not accepted
- ipaPy2 requires Python 3.9 or higher

## Evidence

- [methods] The first step of the IPA pipeline consists in the mapping of the isotope patterns within the dataset considered. This is achieved through the map_isotope_patterns(): "The first step of the IPA pipeline consists in the mapping of the isotope patterns within the dataset considered. This is achieved through the map_isotope_patterns()"
- [abstract] isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations.: "isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations."
- [readme] ipaPy2 requires Python 3.9 or higher: "ipaPy2 requires Python 3.9 or higher"
- [methods] the ipaPy2 library provides a function (clusterFeatures()) able to perform such step: "the ipaPy2 library provides a function (clusterFeatures()) able to perform such step"
