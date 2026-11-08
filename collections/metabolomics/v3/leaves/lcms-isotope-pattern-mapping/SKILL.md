---
name: lcms-isotope-pattern-mapping
description: Use when an LC-MS untargeted feature table (ids, rel.ids, mzs, RTs, Ints)
  has been feature-clustered and isotope peaks must be grouped into isotope-pattern
  units before IPA annotation.
when_to_use_negative:
- Input is an unclustered feature table — run clusterFeatures() first, since map_isotope_patterns()
  assumes already-clustered MS1 features.
- The goal is adduct enumeration or annotation — map_isotope_patterns() is only the
  first step; use compute_all_adducts() and MS1annotation()/MSMSannotation() instead.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: ipaPy2
  role: Supplies the map_isotope_patterns() function used to group isotope peaks into
    isotope fingerprints on the clustered MS1 dataframe.
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Used to load the clustered MS1 feature dataframe (df) and to save the annotated
    isotope_patterns.csv output.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:3c5f81fabe04902751daf3cf9683a59393079c70d9aeab0fb1742066cffa7af2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/francescodc87/ipaPy2
  tools:
  - Python
  - ipaPy2
  - pandas
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lcms-isotope-pattern-mapping

## Summary

Maps MS1 isotope peaks into per-metabolite isotope fingerprints using ipaPy2's map_isotope_patterns(), reducing the combinatorial burden of adduct/isotope enumeration downstream in the IPA Bayesian annotation pipeline.

## When to use

Use when an LC-MS untargeted feature table (ids, rel.ids, mzs, RTs, Ints) has been feature-clustered and isotope peaks must be grouped into isotope-pattern units before IPA annotation.

## When NOT to use

- Input is an unclustered feature table — run clusterFeatures() first, since map_isotope_patterns() assumes already-clustered MS1 features.
- The goal is adduct enumeration or annotation — map_isotope_patterns() is only the first step; use compute_all_adducts() and MS1annotation()/MSMSannotation() instead.

## Inputs

- pandas dataframe of clustered MS1 features with columns ids, rel.ids, mzs, RTs, Int (e.g. ExampleDatasets/README/df_test_pos.csv)
- ionisation mode parameter (1 for positive, -1 for negative)

## Outputs

- dataframe augmented in place with relationship column (bp, bp|isotope, potential bp, potential bp|isotope)
- isotope pattern column (isotope grouping identifier)
- charge column with values within -5 to 5

## How to apply

Run map_isotope_patterns() on the clustered MS1 dataframe immediately after feature clustering (e.g. via clusterFeatures()) and before computing all adducts or calling MS1annotation()/MSMSannotation(). Load the table with pandas from a CSV such as ExampleDatasets/README/df_test_pos.csv containing the ids, rel.ids, mzs, RTs, and Int columns, then call ipa.map_isotope_patterns(df, ionisation=1) in positive mode. The function's defaults (isoDiff=1, ppm=100, MinIsoRatio=0.5) control the isotope mass difference, mass tolerance, and minimum isotope ratio, and it modifies the dataframe in place. It adds and populates the relationship, isotope pattern, and charge columns; the rationale is that grouping isotope peaks into isotope fingerprints speeds up the subsequent Bayesian annotation calculations.

## Related tools

- **ipaPy2** (Supplies the map_isotope_patterns() function used to group isotope peaks into isotope fingerprints on the clustered MS1 dataframe.) — https://github.com/francescodc87/ipaPy2
- **pandas** (Used to load the clustered MS1 feature dataframe (df) and to save the annotated isotope_patterns.csv output.)

## Examples

```
import pandas as pd; from ipaPy2 import ipa; df = pd.read_csv('ExampleDatasets/README/df_test_pos.csv'); df = ipa.map_isotope_patterns(df, ionisation=1); df.to_csv('isotope_patterns.csv')
```

## Evaluation signals

- isotope_patterns.csv exists after running the function
- relationship column contains only the allowed labels: bp, bp|isotope, potential bp, potential bp|isotope
- charge column values fall within -5 to 5
- isotope pattern column is populated (non-empty) for rows classified as isotopes, grouping them with their base peak
- upstream precondition verified: dataframe was previously clustered (df_test_pos.csv rather than df_test_pos_not_clustered.csv)

## Limitations

- Default parameters (isoDiff=1, ppm=100, MinIsoRatio=0.5) may not suit every instrument; the article directs users to the function's help for a detailed description rather than giving tuning guidance.
- The function modifies the dataframe in place, so the caller must explicitly save the result (e.g. isotope_patterns.csv) to avoid losing the mapping.
- Only valid on already-clustered MS1 features; it is not a substitute for feature clustering.

## Evidence

- [abstract] isotope peaks are integrated into isotope fingerprints instead of being treated as individual features, greatly speeding up calculations: "isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations."
- [methods] map_isotope_patterns is the first step of the IPA pipeline: "The first step of the IPA pipeline consists in the mapping of the isotope patterns within the dataset considered. This is achieved through the map_isotope_patterns()"
- [methods] map_isotope_patterns is called on the clustered MS1 dataframe loaded via pandas: "df1=pd.read_csv('ExampleDatasets/README/df_test_pos.csv')"
- [methods] clusterFeatures must precede isotope mapping: "the ipaPy2 library provides a function (clusterFeatures()) able to perform such step, starting from a dataframe containing the measured intensities across several samples"
