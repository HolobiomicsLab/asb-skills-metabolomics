---
name: ms2-spectral-library-matching
description: Use when untargeted metabolomics LC-MS/MS data (e.g., xcms peak-derived
  spectra or MZmine-exported MGF files) needs compound-level MS2 annotation against
  a reference spectral library such as HMDB.
when_to_use_negative:
- When only precursor m/z (MS1) information is available and no fragment spectra exist
  — use matchMz with Mass2MzParam instead; with m/z alone most peaks match several
  possible compounds
- When the reference database provides precursor m/z and a stricter precursor-m/z
  filter is desired — requirePrecursor = FALSE is specific to HMDB which lacks precursor
  m/z
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetaboAnnotation
  role: Provides matchSpectra and CompareSpectraParam for comparing query MS2 spectra
    against reference library spectra with a normalized dot-product score
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: Spectra
  role: Represents query and target spectra; supports import from MGF, MSP, mzML,
    mzXML, netCDF and SQL database backends, plus filterIntensity and addProcessing
    for spectral cleaning
  repo: https://rformassspectrometry.github.io/Spectra/
- name: CompoundDb
  role: Creates and manages the reference compound/spectral database (CompDb) built
    from HMDB 5.0 that supplies the target MS2 spectra
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: xcms
  role: chromPeakSpectra extracts the MS2 spectra for annotated chromatographic peaks
    from the xcms result object as query spectra
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MsBackendMgf
  role: Imports MZmine- or MS-DIAL-exported MGF files as the query Spectra object,
    retaining fields such as FEATURE_ID
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:75cad2459fb7dd415d175d56dc5543bd29188fc98751898cbace76268c44e293
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - MetaboAnnotation
  - MsBackendMgf
  - R
  - Spectra
  - curl
  - xcms
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# ms2-spectral-library-matching

## Summary

Annotates experimental MS2 fragment spectra by comparing them against a reference spectral library (e.g., an HMDB 5.0-derived CompoundDb CompDb) using matchSpectra from the MetaboAnnotation R package with normalized dot-product similarity scoring.

## When to use

Use when untargeted metabolomics LC-MS/MS data (e.g., xcms peak-derived spectra or MZmine-exported MGF files) needs compound-level MS2 annotation against a reference spectral library such as HMDB.

## When NOT to use

- When only precursor m/z (MS1) information is available and no fragment spectra exist — use matchMz with Mass2MzParam instead; with m/z alone most peaks match several possible compounds
- When the reference database provides precursor m/z and a stricter precursor-m/z filter is desired — requirePrecursor = FALSE is specific to HMDB which lacks precursor m/z

## Inputs

- xcms result object with chromatographic peaks (e.g., HighIS_Mix07_CE20_POS.mzML from the 15-standard mix)
- MZmine-exported GNPS MGF file (e.g., MZmine-GNPS_AG_test_GNPS.mgf)
- HMDB 5.0 CompoundDb reference database (CompDb.Hsapiens.HMDB.5.0.sqlite)

## Outputs

- Spectra object of cleaned, intensity-scaled query MS2 spectra
- MS2 match table (e.g., ms2_spectra_matches.csv or mzmine_ms2_spectra_matches.csv) with query identifier, target_compound_id, target_name, and score columns

## How to apply

Extract MS2 spectra for chromatographic peaks from the xcms result object with chromPeakSpectra(return.type = "Spectra", peaks = pks_match$peak_id), then clean them by removing spectral peaks below 5% of each spectrum's base peak intensity via filterIntensity and dropping spectra with fewer than 3 peaks. Scale peak intensities within each spectrum to 0-100 using addProcessing with a custom scaling function. Build the target Spectra object by wrapping the HMDB 5.0 CompDb (CompDb.Hsapiens.HMDB.5.0.sqlite, 217,776 compounds and 64,920 MS2 spectra) in Spectra(cdb), or import MZmine/MS-DIAL exported MGF files as query spectra with Spectra(file, source = MsBackendMgf()). Run matchSpectra with CompareSpectraParam(ppm = 50, requirePrecursor = FALSE) — the precursor filter is disabled because HMDB does not provide precursor m/z — so matches are those with normalized dot-product similarity above the default 0.7 threshold. Extract the match table with spectraData/matchedData selecting columns such as peak_id/FEATURE_ID, target_compound_id, target_name and score, and write it to a CSV.

## Related tools

- **MetaboAnnotation** (Provides matchSpectra and CompareSpectraParam for comparing query MS2 spectra against reference library spectra with a normalized dot-product score) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **Spectra** (Represents query and target spectra; supports import from MGF, MSP, mzML, mzXML, netCDF and SQL database backends, plus filterIntensity and addProcessing for spectral cleaning) — https://rformassspectrometry.github.io/Spectra/
- **CompoundDb** (Creates and manages the reference compound/spectral database (CompDb) built from HMDB 5.0 that supplies the target MS2 spectra) — https://rformassspectrometry.github.io/CompoundDb/
- **xcms** (chromPeakSpectra extracts the MS2 spectra for annotated chromatographic peaks from the xcms result object as query spectra) — https://rformassspectrometry.github.io/Spectra/
- **MsBackendMgf** (Imports MZmine- or MS-DIAL-exported MGF files as the query Spectra object, retaining fields such as FEATURE_ID)

## Examples

```
s <- matchSpectra(std_spectra, Spectra(cdb), param = CompareSpectraParam(ppm = 50, requirePrecursor = FALSE))
```

## Evaluation signals

- Output CSV exists and is parseable with the required columns (peak_id/FEATURE_ID, target_compound_id, target_name, score)
- All reported match scores fall in [0, 1] (normalized dot product, default threshold 0.7)
- The first query spectrum of the 15-standard mix matches Caffeine in the HMDB reference library as shown in the tutorial
- The imported MGF Spectra object retains the FEATURE_ID metadata variable from the original MGF fields
- Cleaned query spectra contain at least 3 spectral peaks with intensities scaled to 0-100

## Limitations

- HMDB does not provide precursor m/z, so requirePrecursor must be set to FALSE, which admits matches without precursor-m/z agreement
- Using only m/z values, most chromatographic peaks match several possible compounds; MS2 matching and retention-time consideration improve confidence
- The similarity threshold is the matchSpectra default of 0.7 (normalized dot product); no per-dataset threshold tuning is described

## Evidence

- [abstract] followed by an MS2-based annotation in which experimental fragment spectra are compared against a reference library.: "followed by an MS2-based annotation in which experimental fragment spectra are compared against a reference library."
- [methods] Such annotation can be performed using the `matchSpectra` from the `MetaboAnnotation` package: "Such annotation can be performed using the `matchSpectra` from the `MetaboAnnotation` package"
- [methods] Remove peaks with an intensity below 5% of BPI: "#' Remove peaks with an intensity below 5% of BPI
std_spectra <- filterIntensity(std_spectra, intensity = low_int)"
- [methods] As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [abstract] The ecosystem supports data from a variety of formats, including, but not limited to, MSP, MGF, mzML, mzXML, netCDF as well as MassBank text files and SQL databases.: "The ecosystem supports data from a variety of formats, including, but not limited to, MSP, MGF, mzML, mzXML, netCDF as well as MassBank text files and SQL databases."
- [readme] This repository provides use cases and examples for the annotation of non-target metabolomics or small compound MS data using the `MetaboAnnotation` and `MetaboCoreUtils` R packages.: "This repository provides use cases and examples for the annotation of non-target metabolomics or small compound MS data using the `MetaboAnnotation` and
`MetaboCoreUtils` R packages."
