---
name: lcms-raw-file-format-conversion
description: Use when raw LC-MS/MS files (e.g. Orbitrap .raw) must be converted to
  mzXML so the GlyKAn AZ app's MS1 Peak ID / MS2 Peak Confirmation tabs can ingest
  them.
when_to_use_negative:
- Data are already in mzXML (or another GlyKAn AZ-compatible format) — conversion
  is redundant
- The analysis target is O-linked glycans or de novo glycan identification — GlyKAn
  AZ's scope is N-linked glycans present in its database
edam_operation: http://edamontology.org/operation_2409
tools:
- name: ProteoWizard
  role: Converts vendor raw Orbitrap LC-MS/MS files to mzXML format for ingestion
    by GlyKAn AZ
- name: GlyKAn AZ
  role: Downstream MATLAB app whose MS1/MS2 tabs consume the converted mzXML files
    for N-linked glycan identification
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:a7c19d0b16bf461510dfc9017aea3c01562b67df8dbcab5095bb479fd7db3990
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  - OrbiTrap
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lcms-raw-file-format-conversion

## Summary

Convert vendor-native Orbitrap LC-MS/MS raw files to mzXML with ProteoWizard, the format required by the GlyKAn AZ app for N-linked glycan MS1/MS2 peak identification.

## When to use

Use when raw LC-MS/MS files (e.g. Orbitrap .raw) must be converted to mzXML so the GlyKAn AZ app's MS1 Peak ID / MS2 Peak Confirmation tabs can ingest them.

## When NOT to use

- Data are already in mzXML (or another GlyKAn AZ-compatible format) — conversion is redundant
- The analysis target is O-linked glycans or de novo glycan identification — GlyKAn AZ's scope is N-linked glycans present in its database

## Inputs

- Vendor-native LC-MS/MS raw files from an Orbitrap ESI-MS instrument (or MALDI-TOF MS data)
- Downstream (post-conversion): MS1 Database with glycan names, theoretical masses (Da), optional expected RTs (min); MS2 Fragment Database; MS2 fragment image repository

## Outputs

- mzXML files loadable into the GlyKAn AZ MS1 Peak ID and MS2 Peak Confirmation tabs
- Subsequently: MS1 output Excel file of candidate glycan peaks and annotated/labeled MS2 fragmentation spectra

## How to apply

Run ProteoWizard (msconvert) on the vendor raw Orbitrap files to produce mzXML, since GlyKAn AZ only accepts mzXML as its MS input. Load the resulting mzXML together with the MS1 Database (glycan names, theoretical masses in Da, optional expected RTs in minutes) into the app's MS1 tab, then chain the MS1 output Excel file, the MS2 Fragment Database, and the MS2 fragment image repository into the MS2 Peak Confirmation tab. Validate the conversion by confirming that the app's MS1 mass search at 25% relative intensity and 10 ppm mass accuracy reproduces known identifications (e.g. 20 true positives at the same RTs as manual analysis for the mAb dataset).

## Related tools

- **ProteoWizard** (Converts vendor raw Orbitrap LC-MS/MS files to mzXML format for ingestion by GlyKAn AZ)
- **GlyKAn AZ** (Downstream MATLAB app whose MS1/MS2 tabs consume the converted mzXML files for N-linked glycan identification) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application

## Examples

```
msconvert sample_orbitrap.raw --mzXML --outdir mzxml/
```

## Evaluation signals

- Output file is valid mzXML and loads without error in the GlyKAn AZ MS1 tab
- MS1 mass search on the converted file at 25% relative intensity / 10 ppm mass accuracy recovers all 20 manually identified glycans with 0 false positives
- Retention times of identified glycan peaks match those from manual analysis (e.g. isomer peaks at RT 13.39, 13.77, 14.07 min for A2G0M4F / A1G1F(α1,6) / A1G1F(α1,3))

## Limitations

- GlyKAn AZ cannot process multiple mzXML files simultaneously; conversion/analysis must be run one file at a time
- Converted files only support the app's N-linked, database-restricted identification workflow (no O-linked or de novo identification)

## Evidence

- [intro] ProteoWizard is used to convert the raw MS files to mzXML format.: "ProteoWizard is used to convert the raw MS files to mzXML format."
- [abstract] This app accepts both OrbiTrap and MALDI-TOF MS data: "This app accepts both OrbiTrap and matrix-assisted laser desorption/ionization-time of flight (MALDI-TOF) MS data"
- [intro] The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da]): "The mzXML file and the MS1 Database containing glycan names, theoretical masses (in daltons [Da])"
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
