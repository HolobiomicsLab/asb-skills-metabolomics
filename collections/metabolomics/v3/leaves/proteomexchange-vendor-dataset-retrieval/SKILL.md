---
name: proteomexchange-vendor-dataset-retrieval
description: Use when a benchmark or conversion study needs original vendor RAW files,
  and the article supplies ProteomeXchange/MetaboLights accession IDs (e.g. PXD021390,
  PXD018139, IPX0001509000, MTBLS2119, XCMS1197236) that must be fetched.
when_to_use_negative:
- The desired input is already an open format (mzML, mz5, mgf or .aird) and no vendor
  RAW baseline is needed for the comparison.
- Only derived peak lists or a processed data matrix is required, not the original
  spectrometry files.
- The dataset is under an acquisition mode Aird currently does not support (ScanningSWATH,
  Ion Mobility), since the article notes these are 'formats under development'.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AirdPro
  role: GUI client whose 'Remote Repository Async Tool' supports batch download from
    ProteomeXchange and MetaboLights before vendor-to-Aird conversion.
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdPro
  role: Provides direct access to vendor RAW files via the integrated ProteoWizard
    SDK for subsequent conversion; the download step supplies the files this tool
    consumes.
  repo: https://github.com/CSi-Studio/AirdPro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:d155f003ca0d9d68ad963d2930106e7ce5af856c941b233b3545c8b44381edf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MSConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# proteomexchange-vendor-dataset-retrieval

## Summary

Download vendor (RAW) mass spectrometry datasets from ProteomeXchange and MetaboLights using the identified accessions (PXD/IPX/MTBLS/XCMS IDs), so that downstream format-conversion and file-size comparisons (e.g. Aird vs mzML vs mgf vs mz5) can be run on the same original files.

## When to use

Use when a benchmark or conversion study needs original vendor RAW files, and the article supplies ProteomeXchange/MetaboLights accession IDs (e.g. PXD021390, PXD018139, IPX0001509000, MTBLS2119, XCMS1197236) that must be fetched.

## When NOT to use

- The desired input is already an open format (mzML, mz5, mgf or .aird) and no vendor RAW baseline is needed for the comparison.
- Only derived peak lists or a processed data matrix is required, not the original spectrometry files.
- The dataset is under an acquisition mode Aird currently does not support (ScanningSWATH, Ion Mobility), since the article notes these are 'formats under development'.

## Inputs

- ProteomeXchange accession IDs (PXD*, IPX*) from the article's dataset table
- MetaboLights accession IDs (MTBLS*)
- XCMS repository ID (XCMS1197236) or equivalent vendor dataset identifiers
- Expected acquisition mode and instrument metadata per accession (DDA, DIA/SWATH, PRM)

## Outputs

- Local copies of the original vendor RAW files (.wiff/.d for SCIEX/Agilent, .raw for Thermo) for each accession
- A mapping of accession -> instrument, acquisition mode, and vendor file path for downstream conversion steps

## How to apply

Collect the vendor file identifiers from the article's discussion table: PXD021390 (TripleTOF 5600, DDA), PXD018139 (Q-Exactive, DIA), IPX0001509000 (Q-Exactive HF, DDA), MTBLS2119 (Q-Exactive, DDA metabolomics), XCMS1197236 (TripleTOF 6600, DDA metabolomics), plus the other listed accessions covering SCIEX, Thermo and Agilent instruments. Query ProteomeXchange for PXD/IPX accessions and MetaboLights for MTBLS accessions to obtain the native RAW file links, or use AirdPro's built-in Remote Repository Async Tool which was designed for these two repositories. Verify that the retrieved files match the instrument and acquisition mode (DDA vs DIA/SWATH) recorded for each accession, since the paper explicitly notes the datasets span proteomics and metabolomics samples on different mass spectrometers. Keep the raw vendor files as the canonical baseline; subsequent conversion steps (MSConvert for mzML/mgf/mz5, AirdPro for .aird) take these RAW files as input.

## Related tools

- **AirdPro** (GUI client whose 'Remote Repository Async Tool' supports batch download from ProteomeXchange and MetaboLights before vendor-to-Aird conversion.) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdPro** (Provides direct access to vendor RAW files via the integrated ProteoWizard SDK for subsequent conversion; the download step supplies the files this tool consumes.) — https://github.com/CSi-Studio/AirdPro

## Examples

```
Invoke AirdPro (GUI) → use the 'Remote Repository Async Tool' panel to add PXD021390, PXD018139, IPX0001509000, MTBLS2119 and XCMS1197236, and let it fetch the vendor RAW files into a local folder.
```

## Evaluation signals

- Each of the seven accessions listed in the paper's discussion table yields a non-empty vendor RAW download; the resulting count matches the 7 datasets reported (e.g. 6 DDA + 1 DIA as described in the source card).
- Retrieved file paths match the accession-to-mode mapping: DDA datasets (PXD021390, IPX0001509000, MTBLS2119, XCMS1197236) and the DIA dataset (PXD018139) align with the acquisition mode recorded in the article.
- Vendor files are retrievable from the expected repositories: ProteomeXchange for PXD/IPX IDs and MetaboLights for MTBLS IDs, with no silent fallback to alternate repositories.
- RAW file sizes are large enough that downstream conversion to mzML/mgf/mz5 and .aird produces non-trivial byte counts, consistent with the paper's average file sizes (2.0 GB Aird in DDA, 24.6 GB in DIA).

## Limitations

- The article lists accession IDs but does not provide explicit download URLs or scripts, so retrieval depends on repository availability and API stability.
- AirdPro's Remote Repository Async Tool is a feature of specific releases (V4.0.X and later); older versions lack this capability.
- Only DDA, DIA/SWATH and PRM acquisitions are currently supported by AirdPro; ScanningSWATH and Ion Mobility datasets are explicitly noted as formats under development, so retrieval of those accessions yields files the conversion pipeline cannot yet process.

## Evidence

- [abstract] We have downloaded seven datasets from ProteomeXchange and Metabolights. They are from different SCIEX, Thermo, and Agilent instruments.: "We have downloaded seven datasets from ProteomeXchange and Metabolights. They are from different SCIEX, Thermo, and Agilent instruments."
- [results] Remote Repository Async Tool for ProteomeXchange and MetaboLights: "Remote Repository Async Tool for ProteomeXchange and MetaboLights"
- [results] ScanningSWATH and Ion Mobility are the formats under development.: "ScanningSWATH and Ion Mobility are the formats under development."
- [discussion] DDA-Dataset1DDAProteomicsTripleTOF 5600PXD021390: "DDA-Dataset1DDAProteomicsTripleTOF 5600PXD021390"
- [methods] AirdPro uses the integrated SDK provided by ProteoWizard [9] to read vendor file data for the first step of conversion: "AirdPro uses the integrated SDK provided by ProteoWizard [9] to read vendor file data for the first step of conversion"
