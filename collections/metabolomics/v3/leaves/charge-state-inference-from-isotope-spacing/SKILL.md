---
name: charge-state-inference-from-isotope-spacing
description: Use when an LC-MS untargeted metabolomics feature table (m/z, RT, intensities
  per sample) contains isotope peaks that must be collapsed into isotope fingerprints
  and charged species inferred before probabilistic annotation.
when_to_use_negative:
- Input is a not-yet-clustered feature table such as ExampleDatasets/README/df_test_pos_not_clustered.csv
  — clusterFeatures() must run first
- Downstream annotation is already complete and the dataframe already carries relationship/isotope-pattern/charge
  columns
- Working with MS2 fragmentation spectra alone (MS2data_example.csv) rather than MS1
  feature tables — map_isotope_patterns() operates on MS1 features
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: ipaPy2
  role: Provides the map_isotope_patterns() function that assigns relationship, isotope-pattern,
    and charge columns to the feature dataframe
  repo: https://github.com/francescodc87/ipaPy2
- name: pandas
  role: Reads the clustered feature CSV and holds the in-place-annotated dataframe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad455
    title: 'ipaPy2: Integrated Probabilistic Annotation (IPA) 2.0—an improved Bayesian-based
      method for the annotation of LC–MS/MS untargeted metabolomics data'
  source_content_hash: sha256:09d4e9be9e13b44e664affbd1d9bc7526d675f161621e125be18a0e3c03dcb40
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4864864864864865
n_claims: 37
---

# charge-state-inference-from-isotope-spacing

## Summary

Infers charge states and isotopic relationships by mapping isotope patterns into isotope fingerprints with ipaPy2's map_isotope_patterns(), reducing redundant isotope peaks into per-metabolite units before adduct computation and annotation.

## When to use

Use when an LC-MS untargeted metabolomics feature table (m/z, RT, intensities per sample) contains isotope peaks that must be collapsed into isotope fingerprints and charged species inferred before probabilistic annotation.

## When NOT to use

- Input is a not-yet-clustered feature table such as ExampleDatasets/README/df_test_pos_not_clustered.csv — clusterFeatures() must run first
- Downstream annotation is already complete and the dataframe already carries relationship/isotope-pattern/charge columns
- Working with MS2 fragmentation spectra alone (MS2data_example.csv) rather than MS1 feature tables — map_isotope_patterns() operates on MS1 features

## Inputs

- Clustered MS1 feature dataframe (pandas DataFrame) with columns ids, rel.ids, mzs, RTs, and Int intensities across samples, e.g. ExampleDatasets/README/df_test_pos.csv
- Ionisation mode integer (ionisation=1 for positive mode)

## Outputs

- In-place-modified pandas DataFrame with new relationship, isotope pattern, and charge columns
- Exported isotope_patterns.csv containing isotope-pattern-reduced features

## How to apply

Load the clustered MS1 feature dataframe — e.g. ExampleDatasets/README/df_test_pos.csv, with ids, rel.ids, mzs, RTs, and Int columns — with pandas. Call ipa.map_isotope_patterns(df, ionisation=1) for positive-mode data; the function defaults are isoDiff=1, ppm=100, and MinIsoRatio=0.5. ipaPy2 modifies the dataframe in place, populating relationship, isotope pattern, and charge columns. Because the article states that 'isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations', this step must run before compute_all_adducts() and any MS1annotation()/MSMSannotation() call. The charge column then encodes the inferred charge state, and the relationship column encodes whether a feature is a base peak, an isotope of one, or a potential candidate for either. Validate the output before downstream annotation by confirming only allowed relationship labels appear and charges stay in the expected integer range.

## Related tools

- **ipaPy2** (Provides the map_isotope_patterns() function that assigns relationship, isotope-pattern, and charge columns to the feature dataframe) — https://github.com/francescodc87/ipaPy2
- **pandas** (Reads the clustered feature CSV and holds the in-place-annotated dataframe)

## Examples

```
import ipaPy2.ipa as ipa; df = pd.read_csv('ExampleDatasets/README/df_test_pos.csv'); ipa.map_isotope_patterns(df, ionisation=1); df.to_csv('isotope_patterns.csv')
```

## Evaluation signals

- isotope_patterns.csv exists after the run and the relationship column contains only the allowed labels: bp, bp|isotope, potential bp, potential bp|isotope
- Charge column values are all within -5 to 5
- Isotope peaks of the same metabolite share one isotope-pattern unit rather than being treated as separate features
- Function defaults were applied: isoDiff=1, ppm=100, MinIsoRatio=0.5 (no accidental parameter drift)
- Downstream compute_all_adducts()/MS1annotation() receive a dataframe already carrying the relationship and charge columns

## Limitations

- Requires Python 3.9 or higher to install ipaPy2
- Function operates on already-clustered MS1 features; feeding unclustered tables will misassign isotopes
- Charge inference depends on the ionisation-mode argument (ionisation=1 for positive mode); wrong mode flag produces incorrect charge assignments

## Evidence

- [abstract] isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations: "isotope peaks are no longer treated as individual features but integrated into isotope fingerprints, greatly speeding up the calculations"
- [methods] The first step of the IPA pipeline consists in the mapping of the isotope patterns within the dataset considered: "The first step of the IPA pipeline consists in the mapping of the isotope patterns within the dataset considered. This is achieved through the map_isotope_patterns()"
- [methods] the ipaPy2 library provides a function (clusterFeatures()) able to perform such step, starting from a dataframe containing the measured intensities across several samples: "the ipaPy2 library provides a function (clusterFeatures()) able to perform such step, starting from a dataframe containing the measured intensities across several samples"
- [readme] df1=pd.read_csv('ExampleDatasets/README/df_test_pos.csv'): "df1=pd.read_csv('ExampleDatasets/README/df_test_pos.csv')"
