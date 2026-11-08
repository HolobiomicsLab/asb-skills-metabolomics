---
name: compound-database-querying
description: Use when chromatographic peaks or features from an xcms-processed mzML
  dataset must be annotated against a reference compound library, and a CompDb (e.g.
  HMDB-derived SQLite) is available or must be built.
when_to_use_negative:
- The query spectra are already annotated elsewhere (e.g. a GNPS MGF job such as MZmine-GNPS_AG_test_GNPS.mgf)
  — reuse those results instead of re-querying a local CompDb.
- No reference compound database covers the compounds of interest, since matches are
  restricted to the CompDb contents (HMDB 5.0 in this workflow).
- MS2 spectra are absent for the peaks of interest and no reference retention times
  exist, leaving only ambiguous m/z-only matching.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: CompoundDb
  role: Creates and manages the CompDb reference database (e.g. from HMDB 5.0) that
    is queried
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: MetaboAnnotation
  role: Provides matchSpectra and matchMz to compare experimental spectra/m/z against
    the CompDb reference
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: Spectra
  role: Holds query and target spectra; provides filterIntensity and addProcessing
    for spectral cleaning/scaling before matching
  repo: https://rformassspectrometry.github.io/Spectra/
- name: xcms
  role: Supplies the chromatographic peak result object and chromPeakSpectra to extract
    MS2 spectra for matched peaks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:545a5f6cacad6ca448f8228bdd7bacc66867e90df507406557b236764ea6e9ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CompoundDb
  - MetaboAnnotation
  - R
  - Spectra
  - xcms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# compound-database-querying

## Summary

Query a reference compound database (CompDb) — e.g. built with CompoundDb from HMDB 5.0 containing 217,776 compounds and 64,920 MS2 spectra — to annotate experimental LC-MS/MS features by MS1 m/z (matchMz) and/or MS2 spectral similarity (matchSpectra with normalized dot-product scores above 0.7).

## When to use

Use when chromatographic peaks or features from an xcms-processed mzML dataset must be annotated against a reference compound library, and a CompDb (e.g. HMDB-derived SQLite) is available or must be built.

## When NOT to use

- The query spectra are already annotated elsewhere (e.g. a GNPS MGF job such as MZmine-GNPS_AG_test_GNPS.mgf) — reuse those results instead of re-querying a local CompDb.
- No reference compound database covers the compounds of interest, since matches are restricted to the CompDb contents (HMDB 5.0 in this workflow).
- MS2 spectra are absent for the peaks of interest and no reference retention times exist, leaving only ambiguous m/z-only matching.

## Inputs

- Raw mzML LC-MS/MS data (e.g. HighIS_Mix07_CE20_POS.mzML from MetaboAnnotationTutorials)
- xcms result object with detected/refined chromatographic peaks (centWave + MergeNeighboringPeaksParam(expandRt = 3))
- Spectra object of MS2 peak spectra extracted via chromPeakSpectra
- CompDb reference database, e.g. CompDb.Hsapiens.HMDB.5.0.sqlite built from HMDB 5.0
- Optional tabular reference lists of m/z, retention times or retention indices (e.g. std_ions.txt)

## Outputs

- Spectra object of match results (spectra_match) linking query peaks to target compounds
- Match table with columns peak_id, target_compound_id, target_name, score (written to ms2_spectra_matches.csv)
- m/z-annotated peak/feature tables from matchMz (pks_match)

## How to apply

Build or load a CompDb reference database with CompoundDb (e.g. CompDb.Hsapiens.HMDB.5.0.sqlite containing all HMDB 5.0 compounds), then use MetaboAnnotation for the queries. For MS1-only annotation run matchMz with Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"), tolerance = 0, ppm = 10), or MzRtParam(ppm = 10, toleranceRt = 7) when reference retention times are available. For MS2 annotation, extract peak spectra from the xcms result with chromPeakSpectra(return.type = "Spectra", peaks = pks_match$peak_id), clean them (filterIntensity to remove peaks below 5% of base peak intensity, keep spectra with more than 2 peaks, scale intensities to 0-100), and run matchSpectra with CompareSpectraParam(ppm = 50, requirePrecursor = FALSE) so matches are those with normalized dot-product similarity above the default 0.7 threshold. Rationale: m/z-only matching is ambiguous — most peaks match several possible compounds — so adding RT or MS2 evidence increases annotation confidence.

## Related tools

- **CompoundDb** (Creates and manages the CompDb reference database (e.g. from HMDB 5.0) that is queried) — https://rformassspectrometry.github.io/CompoundDb/
- **MetaboAnnotation** (Provides matchSpectra and matchMz to compare experimental spectra/m/z against the CompDb reference) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **Spectra** (Holds query and target spectra; provides filterIntensity and addProcessing for spectral cleaning/scaling before matching) — https://rformassspectrometry.github.io/Spectra/
- **xcms** (Supplies the chromatographic peak result object and chromPeakSpectra to extract MS2 spectra for matched peaks)

## Examples

```
sps_match <- matchSpectra(std_spectra, cdb, CompareSpectraParam(ppm = 50, requirePrecursor = FALSE))
```

## Evaluation signals

- ms2_spectra_matches.csv exists and contains the required columns peak_id, target_compound_id, target_name, score
- All reported similarity scores fall within [0, 1]
- Matches reflect the normalized dot-product threshold: scores above the default 0.7 cutoff
- The first query spectrum matches Caffeine in the HMDB reference library, as shown in the tutorial
- Every m/z-only annotation that matches multiple compounds is flagged as ambiguous and, where possible, refined with retention-time or MS2 evidence

## Limitations

- m/z-only matching is ambiguous: most peaks are annotated to several possible compounds based on their m/z
- Matches depend entirely on the content of the reference CompDb (HMDB 5.0 here); unlisted compounds cannot be found
- with requirePrecursor = FALSE, spectral matches may include compounds with differing precursor m/z and require manual inspection
- Query spectra with fewer than 3 peaks (after 5%-of-BPI filtering) are dropped and cannot be matched

## Evidence

- [abstract] Such reference databases can be created and managed with the CompoundDb package.: "Such reference databases can be created and managed with the CompoundDb package."
- [methods] HMDB CompDb reference: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] matchSpectra usage: "Such annotation can be performed using the `matchSpectra` from the `MetaboAnnotation` package"
- [methods] m/z ambiguity: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [methods] spectral cleaning before matching: "Remove peaks with an intensity below 5% of BPI"
- [readme] installation of tooling: "The R packages as well as the example files used in this document can be
installed with the R code below"
