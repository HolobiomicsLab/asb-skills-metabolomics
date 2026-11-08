---
name: gc-ms-quantitative-table-matching
description: Use when processing raw GC–TOF-MS chromatograms (e.g. NetCDF/CDF) from
  multi-sample untargeted metabolomics and you need per-compound integrated areas
  that can be validated against peak-picking tables (XCMS centWave ppm=15, peakwidth=c(1,5);
  MetAlign) for the same compounds.
when_to_use_negative:
- Your input is already a processed compound-quantification feature table (e.g. a
  peak table from XCMS or MetAlign alone) and you do not need spectral deconvolution
  or cross-validation against eRah's integrated areas.
- You are working with LC–MS or other non-GC–MS chromatography data where co-elution
  and EI fragmentation are not the dominant problem; eRah's CMLC/OSD deconvolution
  is designed for GC/MS data.
- You need compound identification via the NIST library directly — eRah cannot read
  the NIST library, which can only be read with NIST software; use eRah's MSP export
  or a GMD/MassBank/HMDB library instead.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: erah
  role: Core R package providing the deconvolution (deconvolveComp, CMLC/OSD), alignment
    (alignComp, setAlPar), missing-compound recovery (recMissComp), identification
    (identifyComp, idList) and table export (dataList, alignList) functions used to
    produce the per-compound integrated-area table that is cross-validated.
  repo: https://CRAN.R-project.org/package=erah
- name: XCMS
  role: 'Peak-picking reference: centWave is run (ppm=15, peakwidth=c(1,5)) on the
    same raw files to produce selective mzRT quantitative ion-peak areas that are
    regressed against eRah''s integrated areas per compound.'
- name: MetAlign
  role: Alternative peak-picking reference used in the article's quantitative comparison,
    supplying per-compound peak areas/intensities for the same 25 samples to be regressed
    against eRah's deconvolved areas.
- name: GC-TOF MS
  role: Instrument platform that generates the co-eluting, EI-fragmented raw chromatograms
    the skill operates on (e.g. the serum samples analyzed by GC–TOF MS in the article).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:5d6eab166f7e24bbe839129f99130ece7902092b4ec153b4182df4feec062041
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GC-TOF MS
  - MetAlign
  - R
  - XCMS
  - erah
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gc-ms-quantitative-table-matching

## Summary

Reproduces the eRah workflow that quantifies deconvolved GC–MS compounds as integrated elution-profile areas and cross-validates those values against selective mzRT ion features from XCMS centWave and MetAlign, per compound and per sample.

## When to use

Use when processing raw GC–TOF-MS chromatograms (e.g. NetCDF/CDF) from multi-sample untargeted metabolomics and you need per-compound integrated areas that can be validated against peak-picking tables (XCMS centWave ppm=15, peakwidth=c(1,5); MetAlign) for the same compounds.

## When NOT to use

- Your input is already a processed compound-quantification feature table (e.g. a peak table from XCMS or MetAlign alone) and you do not need spectral deconvolution or cross-validation against eRah's integrated areas.
- You are working with LC–MS or other non-GC–MS chromatography data where co-elution and EI fragmentation are not the dominant problem; eRah's CMLC/OSD deconvolution is designed for GC/MS data.
- You need compound identification via the NIST library directly — eRah cannot read the NIST library, which can only be read with NIST software; use eRah's MSP export or a GMD/MassBank/HMDB library instead.

## Inputs

- Raw GC–TOF-MS chromatograms in NetCDF/CDF format (e.g. the MTBLS321 or gcspikelite spike-in datasets referenced in the article)
- MSP-format mass spectral library readable by eRah (e.g. custom MassBank export with ~500 unique EI GC–TOF spectra; note NIST library can only be read with NIST software)
- Compound database (e.g. GMD, Fiehn, HMDB, or internal database) listing target compounds and their reference spectra
- Optional side tables: XCMS centWave peak areas (ppm=15, peakwidth=c(1,5)) and MetAlign peak areas for the same samples, for cross-validation

## Outputs

- Per-compound table with compound names, matching scores, and integrated elution-profile areas for each sample (obtained via dataList(ex))
- Alignment list of compounds present in every sample (alignList(ex))
- Cross-validation table r2_comparison_results.csv with per-compound R² from regressions of eRah areas vs XCMS areas and eRah areas vs MetAlign areas, one row per compound (33 compounds in the article's Table 1)
- Optionally an MSP file of all spectra found in the experiment, exported for later library matching

## How to apply

1. Load raw GC–TOF-MS files and, if available, an MSP spectral library via importMSP(); the article uses a custom MassBank-derived library with ~500 unique compounds with EI GC–TOF mass spectra. 2. Define a compound database (e.g. from the Golm Metabolome Database (GMD) or HMDB) and run the eRah pipeline: deconvolveComp() for compound deconvolution using compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD), alignComp() to align spectra across samples (after setAlPar()), and recMissComp() for missing compound recovery. 3. Extract the final table with dataList() (or alignList(ex)) — this yields compound names, matching scores, and the integrated area of each compound per sample. 4. Separately process the same raw files with XCMS centWave (ppm=15, peakwidth=c(1,5)) and MetAlign to obtain peak areas for the selective quantitative m/z-retention-time (mzRT) ion features of each compound. 5. For each compound, regress the eRah integrated area values on the XCMS and MetAlign peak areas across all samples (e.g. 25 serum samples: 11 HIAE adolescents and 14 matched controls), extracting R² per comparison, and restrict to the 33 compounds listed in Table 1. 6. Save the per-compound R² table as r2_comparison_results.csv; the article reports R²>0.90 for most compounds.

## Related tools

- **erah** (Core R package providing the deconvolution (deconvolveComp, CMLC/OSD), alignment (alignComp, setAlPar), missing-compound recovery (recMissComp), identification (identifyComp, idList) and table export (dataList, alignList) functions used to produce the per-compound integrated-area table that is cross-validated.) — https://CRAN.R-project.org/package=erah
- **XCMS** (Peak-picking reference: centWave is run (ppm=15, peakwidth=c(1,5)) on the same raw files to produce selective mzRT quantitative ion-peak areas that are regressed against eRah's integrated areas per compound.)
- **MetAlign** (Alternative peak-picking reference used in the article's quantitative comparison, supplying per-compound peak areas/intensities for the same 25 samples to be regressed against eRah's deconvolved areas.)
- **GC-TOF MS** (Instrument platform that generates the co-eluting, EI-fragmented raw chromatograms the skill operates on (e.g. the serum samples analyzed by GC–TOF MS in the article).)

## Examples

```
install.packages('erah'); library(erah); ex <- deconvolveComp(...); ex <- alignComp(ex, setAlPar(...)); ex <- recMissComp(ex); identifyComp(ex); dataList(ex)
```

## Evaluation signals

- The per-compound R² table (r2_comparison_results.csv) contains an entry for each of the 33 Table 1 compounds under both comparisons (eRah vs XCMS and eRah vs MetAlign).
- All reported R² values lie within [0, 1]; spot-check that high-agreement compounds (as described in the article) reproduce the reported strong correlation between eRah and XCMS/MetAlign quantitative results.
- The eRah output table contains exactly the columns promised by the article: compound names, matching scores, and integrated area per sample — no missing or NA rows for samples that passed the alignment step.
- The alignment list (alignList) contains the same set of compounds across all samples; any compounds lost after recMissComp() re-identification must be re-run through identifyComp before regenerating the table.
- Spot-check one or two compounds by hand: the eRah integrated area for a given sample should be within the same order of magnitude as the XCMS/MetAlign peak area for the corresponding selective mzRT feature of that compound in the same sample.

## Limitations

- The article's Supporting Files 1–4 (including the HIAE serum data and supporting methods) are hosted externally at pubs.acs.org, not included in the main text or the package.
- eRah requires mass spectra in non-binary readable format (MSP); the NIST library can only be read with NIST software, so a custom MassBank-derived library or a GMD/HMDB/Fiehn library must be used instead.
- Version numbers are not stated for eRah, XCMS, MetAlign, or ChromaTOF in the article, so exact parameter and version reproduction may require the R session that accompanied the paper.
- Validation scope is limited to serum samples analyzed by GC–TOF MS; the skill's regression-based cross-validation was demonstrated on 25 serum samples (11 HIAE adolescents and 14 controls) and may not transfer to other matrices or instrument types.
- Even for co-eluted or low relative concentration compounds the correlation between deconvolved compounds and selective mzRT features was high, but the article does not provide a hard threshold below which agreement should be considered unacceptable.

## Evidence

- [abstract] erah outputs a table with compound names, matching scores and integrated area of compounds for each sample: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [abstract] erah's quantitative results were compared to centWave, MetAlign, and ChromaTOF: "The quantitative results of eRah are compared to centWave, the peak-picking algorithm implemented in the widely used XCMS package, MetAlign, and ChromaTOF software."
- [abstract] compound deconvolution using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD): "an innovative compound deconvolution process using multivariate analysis techniques based on compound match by local covariance (CMLC) and orthogonal signal deconvolution (OSD)"
- [methods] alignComp and setAlPar are the alignment entry points: "Alignment is executed with `alignComp()`"
- [methods] recMissComp requires re-identification afterwards: "we always have to re-identify the compounds after executing the missing compound recovery step"
- [methods] identification and data-list extraction: "Identification can be executed by identifyComp, and accessed by idList as follows"
- [results] Metabo Lights MTBLS321 raw data availability: [evidence span withheld — anchor status: not_found]
- [results] MassBank custom library with ~500 unique compounds: "contains a set of ∼500 unique compounds with EI GC–TOF mass spectra"
- [results] NIST library cannot be read directly by eRah: "Note that NIST library can only be read with NIST software."
- [other] task workflow regression of eRah areas on XCMS/MetAlign areas: [evidence span withheld — anchor status: not_found]
