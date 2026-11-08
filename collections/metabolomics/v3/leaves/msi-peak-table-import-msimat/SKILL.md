---
name: msi-peak-table-import-msimat
description: Use when starting a mass2adduct adduct-analysis workflow on MALDI-MSI
  data and you need to load exported peak tables or processed Cardinal objects into
  the `msimat` class before calling massdiff, adductMatch, or corrPairsMSI.
when_to_use_negative:
- When the Cardinal object has not yet been peak-binned with peakBin() — the data
  must be pre-processed first before cardinal2msimat() can convert it
- When you need adduct assignment, co-localization statistics, or mass-difference
  histograms directly — this skill only imports/converts data; those analyses require
  downstream mass2adduct functions on the resulting msimat object
- When working with non-MSI spectra or already-processed feature tables not in the
  CSV/triplet/peak-binned formats this importer expects
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mass2adduct
  role: Provides the `msimat()` function that parses exported CSV peak tables into
    the internal msimat class, plus `cardinal2msimat()` for converting peak-binned
    Cardinal objects
  repo: github.com/kbseah/mass2adduct
- name: SCiLS
  role: Source software used to export MSI data as plain-text CSV files for import
- name: MSiReader
  role: Source software whose 'intensity export' function outputs the comma-delimited
    tables imported by msimat()
- name: Cardinal
  role: R MSI package whose MSProcessedImagingExperiment or MSContinuousImagingExperiment
    objects are converted to msimat format via cardinal2msimat()
  repo: http://cardinalmsi.org/
- name: msimunging.pl
  role: Perl script in the package's inst/ subdirectory that converts large CSV files
    to triplet format and filters peaks by nonzero pixel frequency before import
  repo: github.com/kbseah/mass2adduct
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:6e3d2837de28ebdca13a714c08f2a9dbc56059f424297c07ccf23a8746dca4c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - mass2adduct
  - msimunging.pl
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# msi-peak-table-import-msimat

## Summary

Import exported MALDI-MSI peak intensity tables (CSV from SCiLS or MSiReader, or peak-binned Cardinal experiments) into the mass2adduct `msimat` format so pairwise mass-difference and co-localization adduct analyses can run in R.

## When to use

Use when starting a mass2adduct adduct-analysis workflow on MALDI-MSI data and you need to load exported peak tables or processed Cardinal objects into the `msimat` class before calling massdiff, adductMatch, or corrPairsMSI.

## When NOT to use

- When the Cardinal object has not yet been peak-binned with peakBin() — the data must be pre-processed first before cardinal2msimat() can convert it
- When you need adduct assignment, co-localization statistics, or mass-difference histograms directly — this skill only imports/converts data; those analyses require downstream mass2adduct functions on the resulting msimat object
- When working with non-MSI spectra or already-processed feature tables not in the CSV/triplet/peak-binned formats this importer expects

## Inputs

- Plain-text comma-delimited CSV of MSI peak intensities exported from SCiLS or MSiReader (e.g. via the MSiReader 'intensity export' function)
- Triplet-format files (rows, cols, vals, peaks, spots) produced by the msimunging.pl Perl script for very large CSVs
- Cardinal MSProcessedImagingExperiment or MSContinuousImagingExperiment object that has been peak-binned with peakBin() (requires Cardinal v2.2+)
- Plain text file of mass values, one per line, when pixel intensities are not needed
- imzML file, which can be read into Cardinal first

## Outputs

- An R `msimat` object suitable for massdiff(), hist(), adductMatch(), and corrPairsMSI()
- A class check and summary printout of the imported data
- A plot of total intensities per peak (mass spectrum overview)
- An R data.frame of mass values from scan() when importing a simple peak list

## How to apply

Export preprocessed MSI peak data as plain-text comma-delimited CSV from standard MSI software such as SCiLS or MSiReader, then call `msimat("msi.csv", sep=";")` to build the `msimat` object; verify with `class(d)` and `print(d)`, and view the total-intensity spectrum with `plot(d)`. For very large CSV files (several Gb), first run the provided Perl script `msimunging.pl` to convert to a triplet format (`rows`, `cols`, `vals`, `peaks`, `spots`) and optionally filter low-frequency peaks, then import via `msimat()` passing those five files instead of `csv`. If the data lives in Cardinal (v2.2+), it must be pre-processed with `normalize()`, `smoothSignal()`, `reduceBaseline()`, and `peakBin(peaklist)` before conversion with `cardinal2msimat(d_peaks)`; confirm the conversion by comparing `plot(d_peaks)` and `plot(d_msimat)` peak histograms. If only mass values (no pixel intensities) are needed for downstream massdiff computations, simply scan a plain text file of one mass per line.

## Related tools

- **mass2adduct** (Provides the `msimat()` function that parses exported CSV peak tables into the internal msimat class, plus `cardinal2msimat()` for converting peak-binned Cardinal objects) — github.com/kbseah/mass2adduct
- **SCiLS** (Source software used to export MSI data as plain-text CSV files for import)
- **MSiReader** (Source software whose 'intensity export' function outputs the comma-delimited tables imported by msimat())
- **Cardinal** (R MSI package whose MSProcessedImagingExperiment or MSContinuousImagingExperiment objects are converted to msimat format via cardinal2msimat()) — http://cardinalmsi.org/
- **msimunging.pl** (Perl script in the package's inst/ subdirectory that converts large CSV files to triplet format and filters peaks by nonzero pixel frequency before import) — github.com/kbseah/mass2adduct

## Examples

```
library(mass2adduct); d <- msimat("msi.csv", sep=";"); class(d); plot(d)
```

## Evaluation signals

- class(d) returns "msimat" after import
- print(d) reports a coherent summary of the imported dataset (peaks, spots, values)
- For Cardinal conversions, plot(d_peaks) and plot(d_msimat) show the same peak histogram
- Imported data carries pixel intensities needed by downstream corrPairsMSI() or, if only masses were scanned, is a numeric vector suitable for massdiff(peaks)
- No memory errors when loading very large CSVs — if encountered, re-run msimunging.pl to convert to triplet format before importing

## Limitations

- Very large CSV files (several Gb) may exceed system memory; msimunging.pl itself is still limited by available RAM (e.g. a 40 Gb CSV with 32 Gb RAM will probably not work)
- Cardinal conversion requires Cardinal v2.2 or above and a pre-processed, peak-binned object
- Reading large files directly into R is inefficient and may fail without the triplet-format preprocessing
- Only comma-delimited plain-text tables (or triplet-format components) are supported as direct imports; other formats must be exported first from SCiLS/MSiReader

## Evidence

- [readme] Import exported CSVs via msimat with separator control: [evidence span withheld — anchor status: external_doc]
- [readme] Import MSI intensity data: "d <- msimat("msi.csv", sep=";")
class(d) # "msimat"
print(d) # Report a summary
plot(d) # Mass spectrum of total intensities per peak"
- [readme] Cardinal conversion prerequisites: [evidence span withheld — anchor status: external_doc]
- [readme] Cardinal to msimat conversion call: "# Convert to msimat format for mass2adduct
d_msimat <- cardinal2msimat(d_peaks)"
- [readme] Large file handling via msimunging.pl: [evidence span withheld — anchor status: external_doc]
- [methods] Export requirement from article methods: "This should be exported as plain-text CSV files from standard MSI software such as SCiLS or MSiReader."
- [methods] cardinal2msimat in article methods: "data objects in the `MSProcessedImagingExperiment` or `MSContinuousImagingExperiment` formats can be converted to mass2adduct's `msimat` format with the `cardinal2msimat()` function"
- [methods] imzML as emerging standard input: "Raw MSI data come in different formats, including the emerging standard imzML format, which is based on XML."
