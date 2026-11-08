---
name: spectra-similarity-scoring
description: Use when experimental MS2 spectra extracted from an xcms result (e.g.
  from a mzML file) must be matched against reference fragment spectra from a CompDb/HMDB
  or similar library to assign compound identities to chromatographic peaks.
when_to_use_negative:
- Input is already a feature table or MGF/MSP file where MS2 spectra have not been
  linked to chromatographic peaks in an xcms result object
- The goal is MS1-level annotation by m/z and retention time (use matchMz with Mass2MzParam
  or MzRtParam instead)
- Precursor m/z agreement is required for the annotation (requirePrecursor = FALSE
  here deliberately allows precursor-mismatched matches)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnnotation
  role: Provides matchSpectra and CompareSpectraParam to compute spectral similarity
    between query and reference MS2 spectra
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: Spectra
  role: Spectra object infrastructure for query and reference spectra, filterIntensity,
    and addProcessing for intensity scaling
  repo: https://rformassspectrometry.github.io/Spectra/
- name: CompoundDb
  role: Creates and manages the reference CompDb database (e.g. from HMDB 5.0) supplying
    target MS2 spectra
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: xcms
  role: chromatographic peak detection and chromPeakSpectra extraction of MS2 spectra
    linked to annotated peaks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:d172e5c4f285624063a97686adae71cf9c9734d243329bfc602694e8ef83c994
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CompoundDb
  - MetaboAnnotation
  - R
  - Spectra
  - xcms
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# spectra-similarity-scoring

## Summary

Scores similarity between experimental MS2 fragment spectra and reference library spectra to annotate untargeted metabolomics features, using matchSpectra from the MetaboAnnotation R package against a CompoundDb-derived reference (e.g. HMDB 5.0).

## When to use

Use when experimental MS2 spectra extracted from an xcms result (e.g. from a mzML file) must be matched against reference fragment spectra from a CompDb/HMDB or similar library to assign compound identities to chromatographic peaks.

## When NOT to use

- Input is already a feature table or MGF/MSP file where MS2 spectra have not been linked to chromatographic peaks in an xcms result object
- The goal is MS1-level annotation by m/z and retention time (use matchMz with Mass2MzParam or MzRtParam instead)
- Precursor m/z agreement is required for the annotation (requirePrecursor = FALSE here deliberately allows precursor-mismatched matches)

## Inputs

- xcms result object with detected and refined chromatographic peaks (findChromPeaks/centWave + refineChromPeaks with MergeNeighboringPeaksParam)
- MS2 spectra extracted via chromPeakSpectra from the xcms result
- Reference Spectra object from a CompoundDb CompDb database (e.g. CompDb.Hsapiens.HMDB.5.0.sqlite)
- MS peak list file (e.g. std_ions.txt from MetaboAnnotationTutorials)

## Outputs

- Match table with columns peak_id, target_compound_id, target_name, score (e.g. ms2_spectra_matches.csv)
- Cleaned and intensity-scaled Spectra object of query MS2 spectra

## How to apply

First extract MS2 spectra for chromatographic peaks from the xcms result with chromPeakSpectra(return.type = 'Spectra', peaks = ...), then clean them with filterIntensity to remove peaks below 5% of each spectrum's base peak intensity and drop spectra with fewer than 3 peaks (lengths(std_spectra) > 2). Scale intensities to 0-100 within each spectrum via addProcessing so similarity is computed on comparable signals. Build the target Spectra object from a CompoundDb-created reference (e.g. HMDB 5.0 CompDb with 217,776 compounds and 64,920 MS2 spectra) and run matchSpectra with CompareSpectraParam(ppm = 50, requirePrecursor = FALSE), so matches are those with normalized dot-product similarity above the default 0.7 threshold. Rationale: setting requirePrecursor = FALSE widens matching where precursor m/z may be uncertain, while intensity filtering and scaling prevent low-abundance noise peaks from inflating or deflating similarity scores.

## Related tools

- **MetaboAnnotation** (Provides matchSpectra and CompareSpectraParam to compute spectral similarity between query and reference MS2 spectra) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **Spectra** (Spectra object infrastructure for query and reference spectra, filterIntensity, and addProcessing for intensity scaling) — https://rformassspectrometry.github.io/Spectra/
- **CompoundDb** (Creates and manages the reference CompDb database (e.g. from HMDB 5.0) supplying target MS2 spectra) — https://rformassspectrometry.github.io/CompoundDb/
- **xcms** (chromatographic peak detection and chromPeakSpectra extraction of MS2 spectra linked to annotated peaks)

## Examples

```
matchSpectra(std_spectra, hmdb_spectra, param = CompareSpectraParam(ppm = 50, requirePrecursor = FALSE))
```

## Evaluation signals

- All reported similarity scores fall in [0, 1]
- The output match table exists and contains the required columns peak_id, target_compound_id, target_name, score
- The first query spectrum matches Caffeine in the reference library, as shown in the tutorial
- All retained query spectra have at least 3 spectral peaks after filtering (lengths(std_spectra) > 2)
- No retained spectral peak is below 5% of its spectrum's base peak intensity

## Limitations

- With requirePrecursor = FALSE, matches may be reported even when precursor m/z is inconsistent with the target compound; such matches need manual inspection
- Using only m/z values, most chromatographic peaks match several possible compounds, so similarity scores alone may not uniquely identify a compound
- Requires R >= 4.2 / Bioconductor 3.15 per the MetaboAnnotationTutorials README

## Evidence

- [abstract] MS2-based annotation by comparing experimental fragment spectra against a reference library: "followed by an MS2-based annotation in which experimental fragment spectra are compared against a reference library."
- [methods] matchSpectra performs the annotation: "Such annotation can be performed using the `matchSpectra` from the `MetaboAnnotation` package"
- [methods] Intensity filtering below 5% of BPI: "' Remove peaks with an intensity below 5% of BPI
std_spectra <- filterIntensity(std_spectra, intensity = low_int)"
- [methods] Drop spectra with fewer than 3 peaks: "std_spectra <- std_spectra[lengths(std_spectra) > 2]"
- [methods] m/z alone yields ambiguous annotations: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [readme] README describes the repository purpose and packages: "This repository provides use cases and examples for the annotation of non-target metabolomics or small compound MS data using the `MetaboAnnotation` and `MetaboCoreUtils` R packages."
