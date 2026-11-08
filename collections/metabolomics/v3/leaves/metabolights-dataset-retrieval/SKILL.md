---
name: metabolights-dataset-retrieval
description: Reference for retrieving a named public MS dataset (PXD/IPX/MTBLS/XCMS
  accession) from ProteomeXchange or Metabolights to obtain native vendor files for
  compression or format-conversion experiments.
when_to_use_negative:
- The target data is a ScanningSWATH or Ion Mobility acquisition — the paper states
  these 'are the formats under development' and cannot be converted by AirdPro
- The dataset is not from a supported repository (ProteomeXchange/MetaboLights) or
  not one of the listed PXD/IPX/MTBLS/XCMS accessions — the paper gives no mechanism
  for retrieving data from other sources
- Only already-converted formats (mzML, mgf, mz5) are needed and no raw vendor file
  is required
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0121
tools:
- name: AirdPro
  role: GUI client whose 'Remote Repository Async Tool for ProteomeXchange and MetaboLights'
    is used to download the seven datasets
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Downstream Java SDK for reading the converted Aird files; not used for retrieval
    itself
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:fb1f5186da2dec27610e956077f963aa338ce5a2045b9d0f9c6117871a27ecb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# metabolights-dataset-retrieval

## Summary

Download the seven public MS datasets used in the Aird compression study from ProteomeXchange/Metabolights via the AirdPro Remote Repository Async tool, yielding raw vendor files (SCIEX, Thermo, Agilent) ready for format-conversion benchmarking.

## When to use

Reference for: retrieving a named public MS dataset (PXD/IPX/MTBLS/XCMS accession) from ProteomeXchange or Metabolights to obtain native vendor files for compression or format-conversion experiments.

## When NOT to use

- The target data is a ScanningSWATH or Ion Mobility acquisition — the paper states these 'are the formats under development' and cannot be converted by AirdPro
- The dataset is not from a supported repository (ProteomeXchange/MetaboLights) or not one of the listed PXD/IPX/MTBLS/XCMS accessions — the paper gives no mechanism for retrieving data from other sources
- Only already-converted formats (mzML, mgf, mz5) are needed and no raw vendor file is required

## Inputs

- ProteomeXchange accession (PXD/IPX identifier, e.g. PXD021390, IPX0001509000) or MetaboLights accession (MTBLS identifier, e.g. MTBLS2119)
- AirdPro release binary downloaded from github.com/CSi-Studio/AirdPro/releases/

## Outputs

- Raw vendor mass spectrometry data files (native SCIEX, Thermo, Agilent files) for each retrieved dataset
- Recorded byte sizes of the raw vendor files per dataset (baseline for size_comparison_table.csv)

## How to apply

The Aird paper downloaded seven datasets from ProteomeXchange and Metabolights: DDA-Dataset1 (TripleTOF 5600, PXD021390), DIA-Dataset2 (Q-Exactive, PXD018139), DDA-Dataset3 (Q-ExactiveI, IPX0001509000), DDA-Dataset4 (Q-Exactive metabolomics, MTBLS2119), DDA-Dataset5 (TripleTOF 6600 metabolomics, XCMS1197236), plus PXD002952 and PXD004712 listed in Table 1. The AirdPro client provides a 'Remote Repository Async Tool' that downloads from ProteomeXchange and MetaboLights repositories, with support for vendor file formats like Agilent .d folders. Record the byte size of the raw vendor files, as the downstream comparison computes Aird size relative to RAW (e.g. 2.0 GB average in DDA, 77% of RAW for the DIA dataset). Choose the dataset by acquisition mode (DDA/DIA/SWATH/PRM) and instrument vendor, since Aird supports only DDA, DIA/SWATH, PRM, DIA_PASEF, DDA_PASEF and not ScanningSWATH or Ion Mobility.

## Related tools

- **AirdPro** (GUI client whose 'Remote Repository Async Tool for ProteomeXchange and MetaboLights' is used to download the seven datasets) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Downstream Java SDK for reading the converted Aird files; not used for retrieval itself) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
AirdPro.exe  # launch GUI, then use the Remote Repository Async Tool to fetch PXD021390 / MTBLS2119 / etc.
```

## Evaluation signals

- The retrieved set matches the seven accessions listed in the paper: PXD021390, PXD018139, IPX0001509000, MTBLS2119, XCMS1197236, PXD002952, PXD004712
- Each retrieved file is a native vendor format from SCIEX, Thermo, or Agilent, matching the paper's 'different SCIEX, Thermo, and Agilent instruments'
- Recorded RAW file sizes span the expected ranges: ~3.7 GB average in DDA mode and ~32 GB for the DIA dataset (Aird at 2.0 GB / 24.6 GB respectively, 54% / 77% of RAW)
- The acquisition mode of each dataset (DDA or DIA/SWATH) is correctly identified and recorded, since AirdPro auto-detects acquisition type downstream

## Limitations

- The paper lists only accession IDs; no direct download URLs or retrieval scripts are provided, so retrieval depends on the AirdPro Remote Repository Async tool or manual download
- AirdPro V4.0.X added the Remote Repository Async Tool; earlier versions may not support this function
- AirdPro does not yet support ScanningSWATH or Ion Mobility acquisition methods, so those dataset types cannot be retrieved into Aird format

## Evidence

- [abstract] Seven datasets from SCIEX, Thermo, and Agilent instruments were compared against mzML, mgf, and mz5 formats: "We have downloaded seven datasets from ProteomeXchange and Metabolights. They are from different SCIEX, Thermo, and Agilent instruments."
- [readme] Remote Repository Async Tool for ProteomeXchange and MetaboLights: "Remote Repository Async Tool for ProteomeXchange and MetaboLights"
- [discussion] DDA-Dataset1DDAProteomicsTripleTOF 5600PXD021390: "DDA-Dataset1DDAProteomicsTripleTOF 5600PXD021390"
- [discussion] DDA-Dataset4DDAMetabolomicsQ-ExactiveMTBLS2119: "DDA-Dataset4DDAMetabolomicsQ-ExactiveMTBLS2119"
- [results] ScanningSWATH and Ion Mobility are the formats under development: "ScanningSWATH and Ion Mobility are the formats under development."
