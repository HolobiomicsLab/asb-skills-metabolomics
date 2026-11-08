---
name: xcms-peak-picking-parameterization
description: Use when raw LC-MS mzXML files must be converted into a per-sample feature
  table via embedded XCMS peak picking as the PP baseline of a JPA rescue analysis.
when_to_use_negative:
- The input is an already-aligned feature table — JPA explicitly requires single-sample,
  unaligned csv tables ('Aligned feature table is not acceptable').
- The goal is MS2-recognition (MR) or targeted-list extraction rather than building
  the MS1 PP baseline — use find.level3features() or find.targetfeatures() instead.
- Data are full-scan or DIA acquisitions being routed to the MR step — MR must not
  be used on full-scan/DIA data (though the PP baseline itself still applies).
tools:
- name: JPA
  role: R package whose XCMS.featureTable() function wraps embedded XCMS peak picking
    to produce the PP-labeled feature table and MSnbase object
  repo: https://github.com/HuanLab/JPA
- name: xcms
  role: Embedded peak-picking algorithms invoked via JPA; parameters such as ppm,
    peakwidth, snthresh, prefilter, and noise are passed to XCMS semantics
- name: MSnbase
  role: Data structure in which JPA returns the processed MS data object alongside
    the feature table
- name: devtools
  role: Used to install the JPA package from GitHub (install_github('HuanLab/JPA'))
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:31c775bb8c5781d3b0d18921932e009fb0aab59850f0b98814a16a01b18a2905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA
  tools:
  - JPA
  - MSnbase
  - R
  - devtools
  - xcms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# xcms-peak-picking-parameterization

## Summary

Builds a baseline PP-feature table per mzXML sample by invoking JPA's XCMS.featureTable() with calibrated XCMS peak-picking parameters, so that conventional peak picking (labeled 'PP') can later be augmented with JPA's MS2-recognition (MR) features.

## When to use

Use when raw LC-MS mzXML files must be converted into a per-sample feature table via embedded XCMS peak picking as the PP baseline of a JPA rescue analysis.

## When NOT to use

- The input is an already-aligned feature table — JPA explicitly requires single-sample, unaligned csv tables ('Aligned feature table is not acceptable').
- The goal is MS2-recognition (MR) or targeted-list extraction rather than building the MS1 PP baseline — use find.level3features() or find.targetfeatures() instead.
- Data are full-scan or DIA acquisitions being routed to the MR step — MR must not be used on full-scan/DIA data (though the PP baseline itself still applies).

## Inputs

- Raw LC-MS mzXML file(s) in a single dedicated folder (any vendor format converted via MS-Convert)
- Optional: per-sample csv feature table (mz, rt, rtmin, rtmax, intensity) for custom.featureTable()

## Outputs

- Dataframe featureTable with columns mz, rt, rtmin, rtmax, maxo, sample, level ('PP' for peak-picked features)
- MSnbase object (MSdata) carrying MS2 spectra for downstream MR extraction

## How to apply

Run JPA's XCMS.featureTable(dir=...) on a folder containing only the relevant mzXML files to output a dataframe featureTable plus an MSnbase MSdata object, with each row labeled 'PP' in the 'level' column. Apply the calibrated parameters used in the diluted-urine and NIST-plasma analyses: mz.tol=10, ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, integrate=1, prefilter=c(3,100), noise=100. Each mzXML file yields one sample's feature table; do NOT align samples here — alignment (peak.alignment) is only for multi-sample workflows and the dilution series rescue analysis treats each dilution level as a separate single-sample table. Verify the table structure (columns: mz, rt, rtmin, rtmax, maxo, sample, level) before passing it onward to find.level3features for MR extraction. Alternatively, ingest a per-sample csv feature table via custom.featureTable() when upstream peak picking was done elsewhere; aligned feature tables are explicitly unacceptable as input.

## Related tools

- **JPA** (R package whose XCMS.featureTable() function wraps embedded XCMS peak picking to produce the PP-labeled feature table and MSnbase object) — https://github.com/HuanLab/JPA
- **xcms** (Embedded peak-picking algorithms invoked via JPA; parameters such as ppm, peakwidth, snthresh, prefilter, and noise are passed to XCMS semantics)
- **MSnbase** (Data structure in which JPA returns the processed MS data object alongside the feature table)
- **devtools** (Used to install the JPA package from GitHub (install_github('HuanLab/JPA')))

## Examples

```
featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm = 10, peakwidth = c(5,20), mzdiff = 0.01, snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)
```

## Evaluation signals

- The returned featureTable has exactly the columns mz, rt, rtmin, rtmax, maxo, sample, level, and every row's level is 'PP'.
- The peak-picking call uses the paper's calibrated parameters (ppm=10, peakwidth=c(5,20), mzdiff=0.01, snthresh=6, prefilter=c(3,100), noise=100); deviations change the PP baseline and thus rescue percentages.
- One independent feature table per mzXML file is produced — no cross-sample alignment applied at this stage for single-sample (dilution-series) workflows.
- Downstream rescue computation from the 'level' column should reproduce the paper's benchmarks: ~25% average rescue across dilutions, ~5% in the most dilute samples, and ~40% feature increase for the original-concentration RP(+) sample.
- Retention-time columns are in seconds, as required for any substituted custom csv input.

## Limitations

- Parameter sensitivity: rescue fractions depend on these XCMS settings; PP features 'missed by the conventional peak picking algorithm due to dilution' set the baseline, so different snthresh/noise values shift the MR/PP ratio.
- Aligned or externally aggregated feature tables cannot be ingested — a separate csv per mzXML file is mandatory.
- Input mzXML folder must contain no irrelevant mzXML files, and vendor data must first be converted with MS-Convert.

## Evidence

- [abstract] JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution: "JPA was able to rescue an average of 25% of metabolic features that were missed by the conventional peak picking algorithm due to dilution"
- [readme] readme XCMS.featureTable signature: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object."
- [readme] readme parameter call: "featureTable <- XCMS.featureTable(dir = dir, mz.tol = 10, ppm=10, peakwidth=c(5,20), mzdiff = 0.01,
                             snthresh = 6, integrate = 1, prefilter = c(3,100), noise = 100)"
- [methods] methods raw-data input mode: "Users can also upload their own feature table in csv format generated by other commonly used data processing software"
- [readme] readme alignment note: "Note: for multi-sample analysis, sample alignment is performed after feature extraction."
