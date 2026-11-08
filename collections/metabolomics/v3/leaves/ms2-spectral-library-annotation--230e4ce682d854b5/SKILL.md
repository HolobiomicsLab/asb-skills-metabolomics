---
name: ms2-spectral-library-annotation
description: Use when MS1 features from an LC-MS metabolomics/exposomics run (e.g.,
  urine RP(+) DDA mzXML data) need compound-level identification by matching DDA MS2
  spectra against an in-house .msp standard library.
when_to_use_negative:
- The dataset is full-scan or DIA only, so no MS2 spectra exist to assign to the MS1
  feature table (JPA's MS2-recognition extraction also must be skipped in this case)
- Only a custom (MZmine 2 / MS-DIAL) feature table is available and raw mzXML files
  with MS2 were never acquired, leaving no spectra to match against the .msp library
- No standard spectral library (.msp) or in-house library CSV exists for the compounds
  of interest
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: JPA
  role: Provides ms2.tofeaturetable() to assign MS2 spectra to the MS1 feature table
    and feature.annotation() to match them against the .msp library at dp = 0.1
  repo: github.com/HuanLab/JPA
- name: R
  role: Runtime environment (R >= 4.0.0) in which the JPA annotation functions are
    executed
  repo: github.com/HuanLab/JPA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:9f1f7a3183acf685613121614c03b5d618388960bf3396fa506d65060da13f97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JPA
  - MS-DIAL
  - MZmine 2
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ms2-spectral-library-annotation

## Summary

Assigns MS2 spectra to an MS1 feature table and matches them against a standard .msp spectral library to annotate metabolites, using JPA's ms2.tofeaturetable and feature.annotation functions with a dot-product cutoff (dp = 0.1).

## When to use

Use when MS1 features from an LC-MS metabolomics/exposomics run (e.g., urine RP(+) DDA mzXML data) need compound-level identification by matching DDA MS2 spectra against an in-house .msp standard library.

## When NOT to use

- The dataset is full-scan or DIA only, so no MS2 spectra exist to assign to the MS1 feature table (JPA's MS2-recognition extraction also must be skipped in this case)
- Only a custom (MZmine 2 / MS-DIAL) feature table is available and raw mzXML files with MS2 were never acquired, leaving no spectra to match against the .msp library
- No standard spectral library (.msp) or in-house library CSV exists for the compounds of interest

## Inputs

- Aligned JPA feature table (dataframe with mz, rt, rtmin, rtmax, maxo, sample, level columns) plus its MSnbase object from DDA mzXML files
- Standard metabolite spectral library in .msp format (e.g., convertedLibraryNeg.msp) placed in lib_directory
- In-house targeted metabolite library in CSV format (e.g., LibraryCSVHILIC-.csv) for targeted extraction feeding the annotation step

## Outputs

- Feature table with MS2 spectra assigned to MS1 features and metabolite annotations at dp = 0.1
- Count of annotated metabolites per tool for the sensitivity benchmark against MZmine 2, XCMS, and MS-DIAL

## How to apply

After all feature levels (PP from XCMS.featureTable, MR from find.level3features, target from find.targetfeatures) are extracted and samples are aligned with peak.alignment (bw=5, minfrac=0.5, mzwid=0.015), first run ms2.tofeaturetable to assign MS2 spectra to each MS1 feature. Then call feature.annotation with the directory and name of the .msp library (e.g., convertedLibraryNeg.msp) and a dot-product similarity threshold of dp = 0.1; features whose MS2 spectra match library spectra above this cutoff receive a metabolite annotation. A targeted in-house library CSV (e.g., LibraryCSVHILIC-.csv) can also be used as tarFTname during find.targetfeatures to drive targeted feature extraction. The result is tabulated as the number of annotated metabolites per tool when benchmarking sensitivity against MZmine 2, XCMS, and MS-DIAL.

## Related tools

- **JPA** (Provides ms2.tofeaturetable() to assign MS2 spectra to the MS1 feature table and feature.annotation() to match them against the .msp library at dp = 0.1) — github.com/HuanLab/JPA
- **R** (Runtime environment (R >= 4.0.0) in which the JPA annotation functions are executed) — github.com/HuanLab/JPA

## Examples

```
feature.annotation(featureTable = featureTable, lib_directory = lib_directory, lib_name = "convertedLibraryNeg.msp", dp = 0.1)
```

## Evaluation signals

- Every annotated feature's MS2 match must meet the dot-product threshold dp = 0.1 against the named .msp library (e.g., convertedLibraryNeg.msp)
- The annotated feature table must retain the JPA column schema (mz, rt, rtmin, rtmax, maxo, sample, level) with a level label of PP, MR, or target
- In the benchmark, JPA's annotated-metabolite count must exceed those of MZmine 2, XCMS, and MS-DIAL, consistent with the reported average 2.3-fold more exposure compounds
- The number of annotated metabolites must not exceed the total number of extracted features across PP + MR + target levels
- The .msp library file must load from lib_directory without errors and library retention-time-bearing inputs must use seconds, not minutes

## Limitations

- Annotation quality depends on the coverage of the in-house standard library; features without matching library spectra remain unannotated
- MS2 spectra must be acquired in DDA mode; the MS2-recognition extraction workflow explicitly warns 'Please do not use this function when processing full-scan or DIA data set!'
- Retention-time columns of any imported custom/converted feature tables must be in seconds and ordered as mz, rt, rtmin, rtmax, intensity or the pipeline breaks

## Evidence

- [methods] ms2 assignment before library matching: "The ms2.tofeaturetable() function assigns MS2 spectra to the MS1 feature table."
- [methods] dp threshold invocation: "feature.annotation(featureTable = featureTable, lib_directory = lib_directory, lib_name = lib_name, dp = 0.1)"
- [methods] msp library input: "lib_name <- "convertedLibraryNeg.msp" # name of the library file"
- [readme] annotation as part of JPA scope: "It also performs sample alignment, adduct and metabolite annotations."
- [abstract] benchmark outcome: "JPA detected an average of 2.3-fold more exposure compounds than conventional peak picking only."
