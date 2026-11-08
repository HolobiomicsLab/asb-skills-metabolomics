---
name: chromatographic-peak-picking
description: Use when raw LC-MS mzXML files (any vendor format compatible with MS-Convert)
  need MS1 peak picking to build a JPA feature table, or as the PP baseline step before
  MR and targeted feature extraction in metabolomics/exposomics processing.
when_to_use_negative:
- Input is an already-aligned feature table — JPA requires one unaligned CSV per single
  mzXML file ('Aligned feature table is not acceptable').
- Raw data cannot be converted to mzXML via MS-Convert, since JPA only accepts vendor
  formats compatible with MS-Convert.
- The analysis only needs MS2-recognition or targeted-list extraction without a PP
  base layer — these levels are added to a PP feature table, not generated standalone.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: JPA
  role: R package providing XCMS.featureTable and custom.featureTable for MS1 PP feature
    extraction from raw LC-MS data
  repo: https://github.com/HuanLab/JPA.git
- name: xcms
  role: Embedded XCMS peak picking algorithms used inside JPA's XCMS.featureTable
    to detect chromatographic peaks
- name: devtools
  role: Used to install the JPA package from GitHub before running peak picking
- name: R
  role: Runtime environment (version 4.0.0 or above) in which JPA feature extraction
    is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:22634ae044f3aa6c8cbd62bf9d301af3858b89fd7c850cdfb40a1dd19b035756
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - MS-DIAL
  - MZmine 2
  - R
  - devtools
  - xcms
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# chromatographic-peak-picking

## Summary

Extract MS1 metabolic features ('PP' level) from raw LC-MS mzXML data using JPA's embedded XCMS peak picking functions, producing a dataframe feature table plus an MSnbase object that feeds JPA's MS2-recognition, targeted, alignment, and annotation steps.

## When to use

Use when raw LC-MS mzXML files (any vendor format compatible with MS-Convert) need MS1 peak picking to build a JPA feature table, or as the PP baseline step before MR and targeted feature extraction in metabolomics/exposomics processing.

## When NOT to use

- Input is an already-aligned feature table — JPA requires one unaligned CSV per single mzXML file ('Aligned feature table is not acceptable').
- Raw data cannot be converted to mzXML via MS-Convert, since JPA only accepts vendor formats compatible with MS-Convert.
- The analysis only needs MS2-recognition or targeted-list extraction without a PP base layer — these levels are added to a PP feature table, not generated standalone.

## Inputs

- One or multiple mzXML files from LC-MS analyses (any vendor format compatible with MS-Convert) in a single folder
- Optional: per-sample CSV feature tables (columns in order: m/z, rt, rtmin, rtmax, intensity; retention times in seconds; unaligned), e.g. MZmine 2 output or MS-DIAL CSV converted with convertCSV.R

## Outputs

- Dataframe feature table with columns mz, rt, rtmin, rtmax, maxo, sample, and level (PP features labeled 'PP')
- MSnbase object for downstream JPA steps (find.level3features, peak.alignment, MS2 annotation)

## How to apply

Place all mzXML files of a multi-sample analysis in one folder containing no other irrelevant mzXML files, then call XCMS.featureTable on the directory with pre-optimized parameters such as mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100. The function returns a dataframe feature table (columns mz, rt, rtmin, rtmax, maxo, sample, level with level='PP') and an MSnbase object retained for later MS2-based steps. Alternatively, if feature tables already exist from other software (e.g. MZmine 2 or converted MS-DIAL CSVs via convertCSV.R), supply them with custom.featureTable, which still performs additional XCMS peak picking to add missed PP features. PP extraction is only the first level: after PP, optional find.level3features (MS2 recognition, threshold 2) and find.targetfeatures against an in-house CSV library add non-Gaussian and targeted features, with sample alignment (peak.alignment, bw=5, minfrac=0.5) performed only after all levels are extracted.

## Related tools

- **JPA** (R package providing XCMS.featureTable and custom.featureTable for MS1 PP feature extraction from raw LC-MS data) — https://github.com/HuanLab/JPA.git
- **xcms** (Embedded XCMS peak picking algorithms used inside JPA's XCMS.featureTable to detect chromatographic peaks)
- **devtools** (Used to install the JPA package from GitHub before running peak picking)
- **R** (Runtime environment (version 4.0.0 or above) in which JPA feature extraction is executed)

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm = 10, peakwidth = c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)
```

## Evaluation signals

- Output feature table has exactly the expected columns in order (mz, rt, rtmin, rtmax, maxo, sample) with level='PP' for every extracted feature and retention times in seconds.
- An MSnbase object is returned alongside the dataframe, required by later find.level3features and ms2.tofeaturetable calls.
- With pre-optimized parameters on the urine RP(+) benchmark, the PP layer should form the base of a combined feature table whose total (PP + MR + target) feature and metabolite annotation counts exceed XCMS, MZmine 2, and MS-DIAL alone.
- Custom CSV inputs must contain only the five required columns (m/z, rt, rtmin, rtmax, intensity) with rt edges in seconds; schema violations indicate misformatted input.
- Adding find.level3features (level3.threshold=2) and find.targetfeatures (target.threshold=2) should only append features not already present as PP, so PP count must not decrease after level 3/target extraction.

## Limitations

- Do not run MS2-recognition extraction (find.level3features) when processing full-scan or DIA datasets — it requires DDA MS2 spectra.
- The MR-feature true-positive threshold should be estimated with the thresholdEstimate.R code provided on GitHub rather than guessed.
- Aligned feature tables are not acceptable as custom input; each mzXML file needs its own unaligned CSV.
- Conventional peak picking alone misses features with low signal intensities, poor peak shapes, or parameters mismatch — that is why JPA adds MR and targeted levels on top of PP.

## Evidence

- [methods] PP extraction via embedded XCMS: [evidence span withheld — anchor status: not_found]
- [abstract] JPA combines peak picking with rescue strategies: "JPA performs feature extraction by combining a conventional peak picking algorithm and strategies for (1) recognizing features with bad peak shapes"
- [readme] XCMS.featureTable output format: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [readme] Custom feature tables from other software: "Users can also upload their own feature table in csv format generated by other commonly used data processing software"
- [readme] Unaligned per-sample CSV requirement: "For multi-sample analysis, a CSV feature table for each single mzXML file needs to be provided. Aligned feature table is not acceptable."
